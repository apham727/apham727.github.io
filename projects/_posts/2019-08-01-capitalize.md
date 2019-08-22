---
layout: post
title: Capitalize
description: >
    A budgeting travel service developed during Capital One's Software Engineering Summit Hackathon. 
image: /assets/img/capitalize.png
noindex: true
---


In early August, I got the chance to attend the Capital One Software Engineering Summit
in Clarendon, Virginia. During my time at the summit, I participated in a financial-service themed
hackathon, where my team developed Capitalize, a full-stack web application in Angular + Node that
allows users to find full travel packages within a specified budget. (The link to the github repo for this project can be found [here](https://github.com/apham727/Capitalize){:target="_blank"}).



## Implementation
![Design](/assets/img/overview.png){:class="img-responsive"}
*Capitalize Design Overview*

The frontend of Capitalize was written in Angular, while the backend was written in Node.js + Express. This allowed us to create a webservice on the backend that we could easily call from the frontend with a simple REST call. 

The user would first enter in their budget, category of preference, number of travelers, and length of vacation. The frontend would then compile this information and send it to the backend through the rest call. The backend would then take these parameters and aggregate data from different sites (the Yelp API, the Google Flights API, and the Amadeus API) and return a JSON array to the frontend that contains the full vacation packages as JSON objects. The frontend would then render these in the view. 

## Design Choices and Takeaways
At this point in the web application, we could have created the entire app in Angular, since it would be able to aggregate data and call external APIs itself. However, I thought it would be better to abstract it into a frontend and a backend so that in the future, both could scale more efficiently and independently. 

This application also was a good lesson in passing data between components in Angular. While sharing data between parent and child can be as simple as a *viewChild*, passing data between independent components required the use of services. 