---
layout: post
published: true
title: Isochrones Webmaps Using Leaflet
date: '2016-12-05 02:12:00 +0800'
subtitle: Route 360
---
**Creating an Isochrone Map Using Java Script Route360**
========================

[Route 360](https://www.route360.net/index.html) is a modern open-source JavaScript library designed for mobile-friendly interactive maps which supports both Google and Leaflet maps. The API is free to access (5000 requests per month) and fork on git hub. 

As yet, only [Route 360](https://www.route360.net/index.html) only accepts points as sources, as ploygons would be useful for large destinations such as National Parks. I plan to overlay the Park Boundary with the road network and create *'intersect points'* to use as a measure of *'travel time to entrance'*.  

Motion Intelligence helps business and public organizations harvest the power of advanced spatial analysis. This can be useful in Spatial Planning and business planning. Below, I have adapated a simple polygon map to with travel time and travel type options using Castleton Visitor Centre as an example. I have embeded my [CodePen](http://codepen.io/) and [Gist](https://gist.github.com/DGalexander/6657db41eb3d68c333ad4ebc4007748b). 

<iframe height='471' scrolling='no' title='Peak Gateway' src='//codepen.io/DGAlexander/embed/pNLJGr/?height=471&theme-id=dark&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/DGAlexander/pen/pNLJGr/'>Peak Gateway</a> by David  Alexander (<a href='http://codepen.io/DGAlexander'>@DGAlexander</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

<script src="https://gist.github.com/DGalexander/6657db41eb3d68c333ad4ebc4007748b.js"></script>
