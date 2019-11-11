---
layout: post
published: true
title: Creating Drive Time Polygons
date: '2019-11-08 02:18:00 +0800'
subtitle: Drive time analysis from the Peak District National Park Boundary
---
Motion Intelligence helps business and public organizations harvest the power of advanced spatial analysis. This can be useful in Spatial business planning. 

As yet, [Targomo](https://www.targomo.com/) and [Network Analysis in ESRI](https://doc.arcgis.com/en/arcgis-online/analyze/create-drive-time-areas.htm) only accepts points as sources. Ploygons would be useful for large destinations such as National Parks where entry and exit points to the Park are less well defined. 

# Analysis Steps
I plan to overlay the Park Boundary with the road network and create ‘intersect points’ to use as a measure of ‘travel time to entrance’ coupled with the following point sources;

* PDNPA Cycle Hire Centres
* PDNPA Tourist Information Centres
* PDNPA Car Park Locations
* A road intersections with the PDNP Boundary

A catchment can be defined, in this case, as the area from which visitors using Peak District National Park area are drawn. In this example we create a Nominal Catchment. These can be misleading, as they are based on a radial distance from a defined point. We attempt to mitigate this by using motion intelligence to accurately portray drive time.

Further analysis
* Scrape the data (where available) from data.gov.uk
* Subset the data based on the hours drive time catchment

# Data Anlysis

## Find Some intersect points using R

First we set up the R Environment and load the initial packages. Then the redefine PDNP boundary shape-file is added.

{% highlight r %}
## Set wd
setwd("~/R Projects/TravelTime")
## Load packages
library("GISTools")
{% endhighlight %}

{% highlight r %}
## Load CLP Layer .shp
PDNP <- readShapePoly("~/R Projects/TravelTime/Boundary/PDNP.shp")

## Create a 1km Buffer
PDNP.buffer <- gBuffer(PDNP, width = 1000)
{% endhighlight %}

Plot the output to take a look at the buffer and CLP boundary to make sure that it is loaded correctly. 

{% highlight r %}
plot(PDNP.buffer, add =T, border = "green")
{% endhighlight %}

![1.png]({{site.baseurl}}/img/Rplot02.png)

Load the OS GB Roads data and clip the PDNP Area. This was downloaded from the Ordnance Survey website and is  is subject to the terms at http://os.uk/opendata/licence Contains Ordnance Survey data © Crown copyright and database right (2019). 

Clip the A roads to the PDNP Buffer so we can analyse the intersection points. Here we use A roads as they are the major arterial roads in the area. It is unlikely, a visitor would access the PDNP area, by car on any other type of road. 

{% highlight r %}
## Load OS GB Roads Data (NN = 100km)
roads_SD <- readShapeLines("~/R Projects/TravelTime/oproad_essh_gb/SD_CLIP.shp")
roads_SE <- readShapeLines("~/R Projects/TravelTime/oproad_essh_gb/SE_CLIP.shp")
roads_SJ <- readShapeLines("~/R Projects/TravelTime/oproad_essh_gb/SJ_CLIP.shp")
roads_SK <- readShapeLines("~/R Projects/TravelTime/oproad_essh_gb/SK_CLIP.shp")
#### Repeat Steps for each OS GB Roads Layer ####
## Logical = A Roads
roads.ARoads <- (roads$class == "A Road")
## Subset the SLDF A roads
roads.ARoads <- roads[roads.ARoads,]
## Spatial Clip to CLP Buffer
CLP.Aroads <- gIntersection(CLP.buffer, roads.ARoads, byid = TRUE)
## Remove the roads data as it is >30mb ;)
rm(roads_SD, roads_SE, roads_SJ, roads_SK)

{% endhighlight %}

Plot the clipped roads data to make sure that it is in the same projection. 

{% highlight r %}
## Merge Data
PDNP.Aroads <- rbind(PDNP.Aroads_SD, PDNP.Aroads_SE, PDNP.Aroads_SJ, PDNP.Aroads_SK)
## Take a look
plot(PDNP.Aroads, add = T)
{% endhighlight %}

![2.png]({{site.baseurl}}/img/Rplot03.png)

## Creating Access Points

Where the A roads intersect the PDNP we will use these as theoretical access points to the PDNP area. As mentioned above, this is the logical transport routing for 'non-immediate locals' who we would expect to follow this network. 

{% highlight r %}
## Find Points where A roads intersect the PDNP Boundary
library(maptools)
library(spatstat)
PDNP.Poly <- as(PDNP, "owin")
PDNP.Aroads.Line <- as(PDNP.Aroads, "psp")
PDNP.Poly.edges <- edges(PDNP.Poly)
xPoints <- crossing.psp(PDNP.Aroads.Line, PDNP.Poly.edges)
xPoints <- xPoints[c(1:58)]
##Plot the access points to view their locations on a map.
plot(xPoints, add = TRUE, col = "red", pch = 20, cex = 3)
{% endhighlight %}

![3.png]({{site.baseurl}}/img/Rplot01.png)

Now we convert these access points from X Y location to Latitude and Longitude; the format used to query Route360 API engine. 

{% highlight r %}
## Define Spatial reference http://spatialreference.org/ref/epsg/osgb-1936-british-national-grid/proj4/
library(proj4)
xPoints <- as.data.frame(xPoints)
proj4string <- "+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +datum=OSGB36 +units=m +no_defs"
pj <- project(xPoints, proj4string, inverse=TRUE)
latlon <- data.frame(lat=pj$y, lon=pj$x)
print(latlon) ## mask this from the output for now
{% endhighlight %}

# Motion Intelligence

## Lets compare Route 360 and ESRI Network Analyst

The [Targomo](https://www.targomo.com/) technology Motion Intelligence provides a simple API for large geographic network analysis, route planning and visualization to power complex geo-applications. The API finds the fastest way between any two locations and shows you how far you can travel in a given time range, so you can quickly make decisions about the accessibility of a location.

We query this for the 4 access points, and return one isochrone .geoJSON file. 

{% highlight r %}
```{bash}
curl --compressed -H 'Content-Type: application/json' --data '{"sources":[{"lat":53.03523,"lng":-1.733398,"id":"p1","tm":{"car":{}}},{"lat":53.11506,"lng":-1.636211,"id":"p2","tm":{"car":{}}},{"lat":53.10728,"lng":-1.631378,"id":"p3","tm":{"car":{}}},{"lat":53.10658,"lng":-1.624562,"id":"p4","tm":{"car":{}}},{"lat":53.1041,"lng":-1.612206,"id":"p5","tm":######Removed53 Examples#######{"serializer":"geojson","buffer":"200","values":[1800]}}' 'https://service.route360.net/britishisles/v1/polygon_post?key=XXXXXXXXXXXXXXXXXXXX' | jq '.data' > drive__half_hour.geojson
```
{% endhighlight %}

Now the .geoJSON is downloaded we can import to Spatial Polygons for manipulation. We add a buffer of 1km as the API access level only buffers to 200m. We plot the catchment boundary. 

{% highlight r %}
## Convert to Spatial Polygons 
library("geojsonio")
Catchment <- geojson_read("drive_hour.geojson", method = "local", what = "sp")
## buffer by an extra 1km (r360 free plan only 200m)
Catchment <- gBuffer(Catchment, width = 1000)
## Change the projection
Catchment <- spTransform(Catchment, proj4string)
{% endhighlight %}

## 15 min 30 min 45 min and 60 min Travel Time in Esri 

![4.png]({{site.baseurl}}/img/Rplot04.png)

The map above shows the graduated coloured isochrones generated by [Network Analysis in ESRI](https://doc.arcgis.com/en/arcgis-online/analyze/create-drive-time-areas.htm) from the same A road intersect points but also included Cycle Hire Centres, Visitor Centres and PDNPA Car Parks. The blue line is the Hours Drive catchment generated by [Targomo](https://www.targomo.com/).

We see that the polygon generated by [Targomo](https://www.targomo.com/) is slightly more generous than the isochrones generated by [Network Analysis in ESRI](https://doc.arcgis.com/en/arcgis-online/analyze/create-drive-time-areas.htm). However, in reality these are a very close fit and match real life experience of travelling to the park. 

# Get the External Data 

{% highlight r %}
# IMD Data First web scraped
IMD_2015 <-geojson_read("https://opendata.arcgis.com/datasets/ee0226d33b62409ba2f2b0f99404190a_0.geojson", what = "sp")
IMD_2015 <- spTransform(IMD_2015, proj4string)

# Subset with the hours drive cos it's massive :)
IMD.HoursDrive <- gIntersection(Catchment, IMD_2015)
# Import as .shp
IMD.HoursDrive <- shapefile("IMD.HoursDrive.shp")
{% endhighlight %}

## Lets Map 

{% highlight r %}
library(leaflet)

## Simplify - too big for hosting
library(rmapshaper)
IMD.HoursDrive <- ms_simplify(IMD.HoursDrive, keep = 0.025)

## Set the data to numeric
IMD.HoursDrive$'IMDDec5' <- as.numeric(IMD.HoursDrive$'IMDDec5')

## Create the color bin
pal <- colorBin("RdYlBu", domain = IMD.HoursDrive$'IMDDec5', n=10)

## Create a Popup
LSOA_popup <- paste0("<strong>LSOA: </strong>",
                     IMD.HoursDrive$'LSOA11NM',
                     "<br><strong>IMD Rank (where 1 is most deprived): </strong>",
                     IMD.HoursDrive$'IMDRank5',
                     "<br><strong>IMD Decile (% Most Deprived): </strong>",
                     IMD.HoursDrive$IMDDec5)

mymap <- leaflet(IMD.HoursDrive) %>%
  setView(lng = -1.7, lat = 53.33, zoom = 9) %>% addTiles() %>%
  addPolygons(stroke = FALSE, fillOpacity = 0.7, smoothFactor = 0.5, 
              fillColor = ~pal(IMD.HoursDrive$IMDDec5), popup = LSOA_popup) %>%
  addLegend("bottomright", pal = pal, values = ~'IMD Decile',
            title = "Decile (10=Least Deprived)",
            opacity = 1)
            
library(htmlwidgets)
saveWidget(widget = mymap, file = 'HoursDriveDeprivation.html')
{% endhighlight %}

Maps :-)

<p> 
<iframe frameborder="0" width="700" height="700" 
        sandbox="allow-same-origin allow-scripts"
        scrolling="no" seamless="seamless"
        src="/files/IMD_HOUR.html">
</iframe>
</p>

