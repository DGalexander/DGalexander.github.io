---
layout: post
published: true
title: 'Drive Time Areas '
---
## How many people live within 1 hours drive of the Peak District? 

Motion Intelligence helps business and public organizations harvest the power of advanced spatial analysis. This can be useful in Spatial Planning and business planning.

As yet, Route 360 and Network Analysis in ESRI only accepts points as sources, as ploygons would be useful for large destinations such as National Parks. I plan to overlay the Park Boundary with the road network and create ‘intersect points’ to use as a measure of ‘travel time to entrance’ coupled with the following point sources;

* PDNPA Cycle Hire Centres
* PDNPA Tourist Information Centres
* PDNPA Car Park Locations

# Analysis Steps
Catchment definition is an important part of site location and marketing decision-making and allows analysts to establish a relationship between supply and demand. A catchment can be defined, in this case, as the area from which visitors using Peak District National Park area are drawn.

What is the purpose of this catchment area analysis?

* Guide the Peak District National Park Authroity in the selection of particular geographic sectors on which to focus special marketing activity

In this example we create a Nominal Catchment. These can be misleading, as they are based on a radial distance from a defined point. We attempt to mitigate this by;

* creating a series of access points (A roads to the CLP area)
* using motion intelligence to accurately portray drive time

## Data Anlysis

# Find Some intersect points using R

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

Load the OS GB Roads data and clip the PDNP Area. This was downloaded from the Ordnance Survey website and is  is subject to the terms at http://os.uk/opendata/licence Contains Ordnance Survey data Â© Crown copyright and database right (2016). 

Clip the A roads to the PDNP Buffer so we can analyse the intersection points. Here we use A roads as they are the major arterial roads in the area. It is unlikely, a visitor would access the PDNP area, by car on any other type of road. 

{% highlight r %}
## Load OS GB Roads Data (NN = 100km)
## Load OS GB Roads Data (NN = 100km) Clipped and combined the NN for the PDNP)
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
# Take a look
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
# Define Spatial reference http://spatialreference.org/ref/epsg/osgb-1936-british-national-grid/proj4/
library(proj4)
xPoints <- as.data.frame(xPoints)
proj4string <- "+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +datum=OSGB36 +units=m +no_defs"
pj <- project(xPoints, proj4string, inverse=TRUE)
latlon <- data.frame(lat=pj$y, lon=pj$x)
print(latlon) ## mask this from the output for now
{% endhighlight %}

## Motion Intelligence

# Lets compare Route 360 and ESRI Network Analyst

The Route 360Â technology by Motion Intelligence provides a simple API for large geographic network analysis, route planning and visualization to power complex geo-applications. The API finds the fastest way between any two locations and shows you how far you can travel in a given time range, so you can quickly make decisions about the accessibility of a location.

We query this for the 4 access points, and return one isochrone .geoJSON file. 

{% highlight r %}
```{bash}
curl --compressed -H 'Content-Type: application/json' --data '{"sources":[{"lat":53.03523,"lng":-1.733398,"id":"p1","tm":{"car":{}}},{"lat":53.11506,"lng":-1.636211,"id":"p2","tm":{"car":{}}},{"lat":53.10728,"lng":-1.631378,"id":"p3","tm":{"car":{}}},{"lat":53.10658,"lng":-1.624562,"id":"p4","tm":{"car":{}}},{"lat":53.1041,"lng":-1.612206,"id":"p5","tm":{"car":{}}},{"lat":53.10449,"lng":-1.609329,"id":"p6","tm":{"car":{}}},{"lat":53.10588,"lng":-1.600127,"id":"p7","tm":{"car":{}}},{"lat":53.11283,"lng":-1.577131,"id":"p8","tm":{"car":{}}},{"lat":53.18929,"lng":-1.615673,"id":"p9","tm":{"car":{}}},{"lat":53.23789,"lng":-1.542348,"id":"p10","tm":{"car":{}}},{"lat":53.2981,"lng":-1.557464,"id":"p11","tm":{"car":{}}},{"lat":53.29792,"lng":-1.558754,"id":"p12","tm":{"car":{}}},{"lat":53.29883,"lng":-1.55689,"id":"p13","tm":{"car":{}}},{"lat":53.30024,"lng":-1.555778,"id":"p14","tm":{"car":{}}},{"lat":53.3297,"lng":-1.56879,"id":"p15","tm":{"car":{}}},{"lat":53.37838,"lng":-1.584866,"id":"p16","tm":{"car":{}}},{"lat":53.50599,"lng":-1.696832,"id":"p17","tm":{"car":{}}},{"lat":53.50643,"lng":-1.697451,"id":"p18","tm":{"car":{}}},{"lat":53.50893,"lng":-1.701016,"id":"p19","tm":{"car":{}}},{"lat":53.50893,"lng":-1.701009,"id":"p20","tm":{"car":{}}},{"lat":53.5086,"lng":-1.700534,"id":"p21","tm":{"car":{}}},{"lat":53.50859,"lng":-1.700523,"id":"p22","tm":{"car":{}}},{"lat":53.54998,"lng":-1.836075,"id":"p23","tm":{"car":{}}},{"lat":53.56791,"lng":-1.84499,"id":"p24","tm":{"car":{}}},{"lat":53.58107,"lng":-1.974792,"id":"p25","tm":{"car":{}}},{"lat":53.58072,"lng":-1.975947,"id":"p26","tm":{"car":{}}},{"lat":53.53461,"lng":-1.974776,"id":"p27","tm":{"car":{}}},{"lat":53.47188,"lng":-1.966514,"id":"p28","tm":{"car":{}}},{"lat":53.44408,"lng":-1.92418,"id":"p29","tm":{"car":{}}},{"lat":53.42851,"lng":-1.945765,"id":"p30","tm":{"car":{}}},{"lat":53.3814,"lng":-1.946886,"id":"p31","tm":{"car":{}}},{"lat":53.3705,"lng":-1.935578,"id":"p32","tm":{"car":{}}},{"lat":53.34308,"lng":-1.922189,"id":"p33","tm":{"car":{}}},{"lat":53.31781,"lng":-1.891723,"id":"p34","tm":{"car":{}}},{"lat":53.31778,"lng":-1.891706,"id":"p35","tm":{"car":{}}},{"lat":53.31164,"lng":-1.884476,"id":"p36","tm":{"car":{}}},{"lat":53.24657,"lng":-1.8701,"id":"p37","tm":{"car":{}}},{"lat":53.22402,"lng":-1.867877,"id":"p38","tm":{"car":{}}},{"lat":53.22399,"lng":-1.8677,"id":"p39","tm":{"car":{}}},{"lat":53.22397,"lng":-1.867544,"id":"p40","tm":{"car":{}}},{"lat":53.20517,"lng":-1.826518,"id":"p41","tm":{"car":{}}},{"lat":53.24134,"lng":-1.941685,"id":"p42","tm":{"car":{}}},{"lat":53.24513,"lng":-1.943028,"id":"p43","tm":{"car":{}}},{"lat":53.27333,"lng":-1.950377,"id":"p44","tm":{"car":{}}},{"lat":53.30054,"lng":-1.973807,"id":"p45","tm":{"car":{}}},{"lat":53.30936,"lng":-1.976442,"id":"p46","tm":{"car":{}}},{"lat":53.31015,"lng":-1.979418,"id":"p47","tm":{"car":{}}},{"lat":53.31242,"lng":-1.985095,"id":"p48","tm":{"car":{}}},{"lat":53.3125,"lng":-1.985307,"id":"p49","tm":{"car":{}}},{"lat":53.26037,"lng":-2.0658,"id":"p50","tm":{"car":{}}},{"lat":53.20343,"lng":-2.080459,"id":"p51","tm":{"car":{}}},{"lat":53.20314,"lng":-2.089357,"id":"p52","tm":{"car":{}}},{"lat":53.20157,"lng":-2.092504,"id":"p53","tm":{"car":{}}},{"lat":53.1424,"lng":-1.981214,"id":"p54","tm":{"car":{}}},{"lat":53.04647,"lng":-1.856001,"id":"p55","tm":{"car":{}}},{"lat":53.04617,"lng":-1.855631,"id":"p56","tm":{"car":{}}},{"lat":53.04616,"lng":-1.855605,"id":"p57","tm":{"car":{}}},{"lat":53.04604,"lng":-1.855336,"id":"p58","tm":{"car":{}}}],"polygon":{"serializer":"geojson","buffer":"200","values":[1800]}}' 'https://service.route360.net/britishisles/v1/polygon_post?key=XXXXXXXXXXXXXXXXXXXX' | jq '.data' > drive__half_hour.geojson
```
{% endhighlight %}

Now the .geoJSON is downloaded we can import to Spatial Polygons for manipulation. We add a buffer of 1km as the API access level only buffers to 200m. We plot the catchment boundary. 

{% highlight r %}
# Convert to Spatial Polygons 
library("geojsonio")
Catchment <- geojson_read("drive_hour.geojson", method = "local", what = "sp")
# buffer by an extra 1km (r360 free plan only 200m)
Catchment <- gBuffer(Catchment, width = 1000)
# Change the projection
Catchment <- spTransform(Catchment, proj4string)
{% endhighlight %}

## 15 min 30 min 45 min and 60 min Travel Time in Esri 

![4.png]({{site.baseurl}}/img/Rplot04.png)

The map above shows the graduated coloured isochrones generated by ESRI Network Analyst from the same A road intersect points but also included Cycle Hire Centres, Visitor Centres and PDNPA Car Parks. The blue line is the Hours Drive catchment generated by Route 360.

We see that the polygon generated by route 360 is slightly more generous than the ESRI polygons. However, in reality these are a very close fit and match real life experience of travelling to the park. 


# Get the External Data 

{% highlight r %}
# IMD Data First
IMD_2015 <-geojson_read("https://opendata.arcgis.com/datasets/ee0226d33b62409ba2f2b0f99404190a_0.geojson", what = "sp")
IMD_2015 <- spTransform(IMD_2015, proj4string)

# Subset with the hours drive cos it's massive :)
IMD.HoursDrive <- gIntersection(Catchment, IMD_2015)
# Import as .shp
IMD.HoursDrive <- shapefile("IMD.HoursDrive.shp")


# Check what it looks like
plot(IMD.HoursDrive, add = T, col = "green")
{% endhighlight %}












