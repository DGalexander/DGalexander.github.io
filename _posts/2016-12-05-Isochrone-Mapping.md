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

Motion Intelligence helps business and public organizations harvest the power of advanced spatial analysis. This can be useful in Spatial Planning and business planning. Below, I have adapated a simple polygon map to with travel time and travel type options using Bakewell Visitor Centre as an example.

<p> 
<iframe frameborder="0" width="700" height="500" 
        sandbox="allow-same-origin allow-scripts"
        scrolling="no" seamless="seamless"
        src="/files/2017-01-11-ISO_MAP.html">
</iframe>
</p>


