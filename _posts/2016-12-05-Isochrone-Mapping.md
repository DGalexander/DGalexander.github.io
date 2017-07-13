---
layout: post
published: true
title: Isochrone Webmaps
date: '2016-12-05 02:12:00 +0800'
subtitle: Isochrones -  using Java Script and Leaflet
---
**Creating an Isochrone Map Using Java Script Route360**
========================

[Route 360](https://www.route360.net/index.html) is a modern open-source JavaScript library designed for mobile-friendly interactive maps which supports both Google and Leaflet maps. The API is free to access (5000 requests per month) and fork on git hub. 

As yet, only [Route 360](https://www.route360.net/index.html) only accepts points as sources, as ploygons would be useful for large destinations such as National Parks. I plan to overlay the Park Boundary with the road network and create *'intersect points'* to use as a measure of *'travel time to entrance'*.  

Motion Intelligence helps business and public organizations harvest the power of advanced spatial analysis. This can be useful in Spatial Planning and business planning. Below, I have adapated a simple bash query around using R as a GIS to show travel time to a Landscape Partnerhip area in Scotland. 

### Introduction

Catchment definition is an important part of site location and marketing decision-making and allows
analysts to establish a relationship between supply and demand. A catchment can be defined, in this case, as the area from which visitors using Callander Landscape Partnership (CLP) area are drawn. 

What is the purpose of this catchment area analysis?

* guide the CLP in the selection of particular geographic sectors on which to focus special marketing activity 
* evaluate, subsequently, the impact made by this special marketing activity 

In this example we create a Nominal Catchment. These can be misleading, as they are based on a radial distance from a defined point. We attempt to mitigate this by;

* creating a series of access points (A roads to the CLP area)
* using motion intelligence to accurately portray drive time

Consideration for further analysis;

* use Postcodes from the Visitor Survey Data to spatially weight areas 'more likely visit the CLP area'
* use Postcodes from the Visitor Survey Data to create a catchment decay profile (Nominal Vs Real Catchment)

## Data Analysis

First we set up the R Environment and load the initial packages. Then the redefine CLP boundary shape-file is added.

First we set up the R Environment and load the initial packages. Then the redefine CLP boundary shape-file is added.

{% highlight r %}
## Set wd
setwd("~/Desktop/CALLANDER/R")
## Load Packages
library("GISTools")
{% endhighlight %}

{% highlight r %}
## Load CLP Layer .shp
CLP <- readShapePoly("~/Desktop/CALLANDER/Boundary/CLP/20170127_CLP_Boundary_Version_2.shp")

## Create a 1km Buffer
CLP.buffer <- gBuffer(CLP, width = 1000)
{% endhighlight %}

Plot the output to take a look at the buffer and CLP boundary to make sure that it is loaded correctly. 

{% highlight r %}
plot(CLP.buffer)
plot(CLP, add = T, border = "blue")
{% endhighlight %}

![1.png]({{site.baseurl}}/img/1.png)

Load the OS GB Roads data and clip the CLA Area. This was downloaded from the Ordnance Survey website and is  is subject to the terms at http://os.uk/opendata/licence Contains Ordnance Survey data Â© Crown copyright and database right (2016). 

Clip the A roads to the CLP Buffer so we can analyse the intersection points. Here we use A roads as they are the major arterial roads in the area. It is unlikely, a visitor would access the CLP area, by car on any other type of road. 

{% highlight r %}
## Load OS GB Roads Data (NN = 100km)
roads <- readShapeLines("~/Desktop/CALLANDER/Boundary/OSOpenRoadsGB/data/NN_RoadLink.shp")
## Logical = A Roads
roads.ARoads <- (roads$class == "A Road")
## Subset the SLDF A roads
roads.ARoads <- roads[roads.ARoads,]
## Spatial Clip to CLP Buffer
CLP.Aroads <- gIntersection(CLP.buffer, roads.ARoads, byid = TRUE)
## Remove the roads data as it is >30mb ;)
rm(roads)
{% endhighlight %}

Plot the clipped roads data to make sure that it is in the same projection. 

{% highlight r %}
plot(CLP.buffer)
plot(CLP, add = T, border = "blue")
plot(CLP.Aroads, add = T, border = "red")
{% endhighlight %}

![2.png]({{site.baseurl}}/img/2.png)

## Creating Access Points

Where the A roads intersect the CLP we will use these as theoretical access points to the CLP area. As mentioned above, this is the logical transport routing for 'non-immediate locals' who we would expect to follow this network. 

{% highlight r %}
## Find Points where A roads intersect the CLP Boundary
library(maptools)
library(spatstat)
CLP.Poly <- as(CLP, "owin")
CLP.Aroads.Line <- as(CLP.Aroads, "psp")
CLP.Poly.edges <- edges(CLP.Poly)
xPoints <- crossing.psp(CLP.Aroads.Line, CLP.Poly.edges)
xPoints <- xPoints[c(1:4)]
{% endhighlight %}

Plot the access points to view their locations on a map.

{% highlight r %}
plot(CLP.buffer)
plot(CLP, add = T, border = "blue")
plot(CLP.Aroads, add = T, border = "red")
plot(xPoints, add = TRUE, col = "red", pch = 20, cex = 3)
{% endhighlight %}

![3.png]({{site.baseurl}}/img/3.png)

Now we convert these access points from X Y location to Latitude and Longitude; the format used to query Route360 API engine. 

{% highlight r %}
# Define Spatial reference http://spatialreference.org/ref/epsg/osgb-1936-british-national-grid/proj4/
library(proj4)
xPoints <- as.data.frame(xPoints)
proj4string <- "+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +datum=OSGB36 +units=m +no_defs"
## proj4string <- "+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +wktext  +no_defs"
# Transform data
pj <- project(xPoints, proj4string, inverse=TRUE)
latlon <- data.frame(lat=pj$y, lon=pj$x)
print(latlon)
{% endhighlight %}

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg .tg-yw4l{vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-yw4l">lat</th>
    <th class="tg-yw4l">long</th>
  </tr>
  <tr>
    <td class="tg-yw4l">56.21222</td>
    <td class="tg-yw4l">-4.137809</td>
  </tr>
  <tr>
    <td class="tg-yw4l">56.29837</td>
    <td class="tg-yw4l">-4.303078</td>
  </tr>
  <tr>
    <td class="tg-yw4l">56.22998</td>
    <td class="tg-yw4l">-4.275506</td>
  </tr>
  <tr>
    <td class="tg-yw4l">56.21646</td>
    <td class="tg-yw4l">-4.222594</td>
  </tr>
</table>

## Motion Intelligence

The Route 360Â°-technology by Motion Intelligence provides a simple API for large geographic network analysis, route planning and visualization to power complex geo-applications. The API finds the fastest way between any two locations and shows you how far you can travel in a given time range, so you can quickly make decisions about the accessibility of a location.

We query this for the 4 access points, and return one isochrone .geoJSON file. 

{% highlight r %}
```{bash}
curl --compressed -H 'Content-Type: application/json' --data '{"sources":[{"lat":56.2122,"lng":-4.137809,"id":"p1","tm":{"car":{}}},{"lat":56.29837,"lng":-4.303078,"id":"p2","tm":{"car":{}}},{"lat":56.22998,"lng":-4.275506,"id":"p3","tm":{"car":{}}},{"lat":56.21646,"lng":-4.222594,"id":"p4","tm":{"car":{}}}],"polygon":{"serializer":"geojson","buffer":"200","values":[3600]}}' 'https://service.route360.net/britishisles/v1/polygon_post?key=OZIWK8M2YYLAMA2VNPAU' | jq '.data' > drive_hour.geojson
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
plot(Catchment)
plot(CLP, add = T, col = "green")
{% endhighlight %}

![4.png]({{site.baseurl}}/img/4.png)

SNS Data Zones, were developed to monitor and develop policy at small area level. Scotland is split in to 6,976 SNS data zones, which have roughly equal population but vary in size. There are 38 indicators to measure the different sides of deprivation in each data zone, like pupil performance, travel times to the GP, crime, unemployment and many others. Focusing on small areas shows the
different issues there are in each neighborhood. These could be poor housing conditions, a lack of skills or good education, or poor public transport.

It is important to note that the SIMD identifies deprived areas not people. 

{% highlight r %}
# Add SNS Data Zones
SNS <- readShapePoly("~/Desktop/CALLANDER/Boundary/SG_SIMD_2016/SG_SIMD_2016.shp", proj4string = CRS(proj4string))
{% endhighlight %}

We then do a spatial query or intersection of the data zones to the catchment boundary.

{% highlight r %}
# Subset SNS layer intersects Catchment
SNS.Catch <- gIntersection(Catchment, SNS, byid = TRUE)

# Replace the concatenation of Catchment
row.names(SNS.Catch) <- gsub("buffer ","", row.names(SNS.Catch))
{% endhighlight %}

Using the row names we re-attach the data to the newly created polygons to create a Spatial Polygons Data frame of the SIMD for analysis.

{% highlight r %}
# Create a DF from subset of SNS data  
SNS.data <- as.data.frame(SNS@data[row.names(SNS.Catch), ])

# Create a SPDF
SNS.Catch <- SpatialPolygonsDataFrame(SNS.Catch, SNS.data)

# Subset the SNS Data for full polygons
SNS <- SNS[row.names(SNS.Catch), ]
{% endhighlight %}

## Exploratory Data Analysis

{% highlight r %}
# Attach a DF
attach(data.frame(SNS.Catch))
{% endhighlight %}

Lets explore some key variables of population. It shows there are just over 3 million people in the catchment area we have defined.

{% highlight r %}
# Let's take a gander at the data :)
# Polulation
sum(SAPE2014)

# Working Age Poulation
sum(WASAPE2014)
{% endhighlight %}

A (very!) quick look at the Deciles suggests that there is an even spread of deprived and non-deprived areas. (1 being most deprived top 10%)

{% highlight r %}
# Histogram of Decile 
table(Decile)
{% endhighlight %}


Employment Rate

{% highlight r %}
# Let's take one of the variables Employment Rate
hist(EmpRate, breaks = 20)
{% endhighlight %}

![5.png]({{site.baseurl}}/img/5.png)

Let's test some mapping!

{% highlight r %}
# Set some cuts
quantileCuts(EmpRate, 5)
rangeCuts(EmpRate, 5)
sdCuts(EmpRate, 5)

# Plot perameters 
par(mar = c(0.25, 0.25, 2, 0.25))
par(mfrow = c(1, 3))
par(lwd = 0.7)

# Quantile Cuts
shades <- auto.shading(EmpRate, cutter = quantileCuts, n = 5, cols = brewer.pal(5, "RdYlGn"))
choropleth(SNS, EmpRate, shading = shades)
title("Quantile Cuts", cex.main = 1)

# Range Cuts
shades <- auto.shading(EmpRate, cutter = rangeCuts, n = 5, cols = brewer.pal(5, "RdYlGn"))
choropleth(SNS, EmpRate, shading = shades)
title("Range Cuts", cex.main = 1)

# Standard Deviation Cuts
shades <- auto.shading(EmpRate, cutter = sdCuts, n = 5, cols = brewer.pal(5, "RdYlGn"))
choropleth(SNS, EmpRate, shading = shades)
title("Standard Deviation Cuts", cex.main = 1)

par(mfrow = c(1,1))
{% endhighlight %}

{% highlight r %}
# Quantile Cuts
shades <- auto.shading(Rank, cutter = rangeCuts, n = 5, cols = brewer.pal(5, "RdYlGn"))
choropleth(SNS,Rank, shading = shades)
title("Quantile Cuts", cex.main = 1)

par(mfrow = c(1,1))
{% endhighlight %}

![6.png]({{site.baseurl}}/img/6.png)

{% highlight r %}
# Quantile Cuts
shades <- auto.shading(Rank, cutter = rangeCuts, n = 5, cols = brewer.pal(5, "RdYlGn"))
choropleth(SNS,Rank, shading = shades)
title("Quantile Cuts", cex.main = 1)
{% endhighlight %}

![7.png]({{site.baseurl}}/img/7.png)













