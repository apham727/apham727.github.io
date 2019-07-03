---
layout: post
title: Route Destination Finder
description: >
  A web application that finds convenient destinations along a route
image: assets/img/routefinder.png
noindex: true
---
The git repo for this project can be found [here](https://github.com/apham727/FindAlongRoute).

When my family and I were driving to Colorado back when I was ten, I remember my mom asking where the nearest gas station was and me thinking that it would be really convenient if Google Maps allowed me to search for gas stations along our current route. That inspired this project, though Google Maps later implemented this feature anyways.

The purpose of this application is to find gas stations, restaurants, etc. along a route within a specified distance from the route. It uses the Python-Flask web framework to serve the application. 

## Challenges
One of the challenges I faced was that the Google Maps API put a query per second limit on calls to its service. I originally tried to query the Maps API for destinations at small, preset increments along the route, but I ran into the aforementioned query limit. As a workaround, I queried the Google Maps API at 20 equally spaced intervals around the route with a radius backwards-calculated from the user-specified distance. I then calculated the shortest distance from any location from the route to ensure it was less than the user-specified distance. 

In this process, I learned about the haversine distance, which is the shortest distance between two objects on a globe. The javascript function I used is shown below: 
~~~javascript
      function haversine(lat1, lon1, lat2, lon2) {
        var R = 6371; // km
        var dLat = toRad(lat2-lat1);
        var dLon = toRad(lon2-lon1);
        var lat1 = toRad(lat1);
        var lat2 = toRad(lat2);

        var a = Math.sin(dLat/2) * Math.sin(dLat/2) +
          Math.sin(dLon/2) * Math.sin(dLon/2) * Math.cos(lat1) * Math.cos(lat2);
        var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        var d = R * c;
        return d;
      }
~~~

## Takeaways
This project was a solid exercise in making workarounds to external factors that were out of my control (or more precisely, that I wasn't willing to pay for). It was also my first introduction to Javascript. 

