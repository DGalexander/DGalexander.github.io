---
layout: post
published: true
title: Infrared Bird Box
subtitle: Raspberry pi birdbox - stream video feed to the internet
date: '2016-07-21 02:18:00 +0800'
---
**Raspberry Pi Infrared Bird box**
========================

This summer I have finally gotten round to building a raspberry pi bird box. It is based on this guide by [Raspberry Pi Foundation](https://www.raspberrypi.org/learning/infrared-bird-box/). Although it is late in the year for any nesting this year, Nestboxes are best put up during the autumn. Many birds will enter nestboxes during the autumn and winter, looking for a suitable place to roost or perhaps to feed. 

The best place to situate my Nestbox will be on a North facing wall out of direct sunlight. There is a small overhang of guttering which should shelter the box and the raspberry pi. The height is about 1.5 meters, hopefully this will attract Blue, Great or Coal Tits. 


## Equipment

 - Raspberry Pi Model A+
 - USB Wifi Adapter for the Raspberry Pi
 - Raspberry Pi NOIR camera module
 - 220 Ohm 1% Resistors
 - 5mm Infrared IR LED Light Emitting Diode
 - Bird box [Gardman](http://www.diy.com/departments/gardman-brown-nest-box/189469_BQ.prd)

<img src="/img/Birdbox_schem.png" alt="BirdBox Schematic" align="left" style="PADDING-RIGHT: 2px"/>

## Method


 1. Set up the Camera Board
 2. Wire up the Infra Red LED
 3. Adjust the camera sighted in the Birdbox
 4. Compile ffmpeg and stream to the internet
 5. Create a script with a `while` loop to constantly send out video
 6. Create an SSH Connection using [Screen](https://en.wikipedia.org/wiki/GNU_Screen) 
 6. Edit the `rc.local`script to run the Screen every time the Pi boots

<img src="/img/IMG_3119.png" alt="BirdBox" align="left" style="PADDING-RIGHT: 0px"/>

# Results

<img src="/img/IMG_3223.png" alt="BirdBox Installed" align="right" style="PADDING-RIGHT: 0px"/>
