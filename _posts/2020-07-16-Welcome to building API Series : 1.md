---
bg: "Headshot2.png"
layout: post
date: 2020-07-16
mathjax: true
title: "Welcome to building API Series - 1"
crawlertitle: "API & Swagger"
categories: posts
tags: ['api', 'Framework', 'swagger', 'world', 'programmer']
author: Yash Karwa
twitterImg: 
excerpt: "Welcome to building API Series"
excerpt-image: '<img src="../../assets/images/Headshot.png" width="220" alt="Welcome to building API Series -1" title="Welcome to building API Series -1">
<em> Welcome to building API Series - 1 </em>'
---

I want to introduce the first contract approach. The contract is just documentation - You need to work with the consumer to define the response.

  
> **Don't write code; work early on the requested response.**

  
The question is, what should be the right contract or documentation should be?

  
One of the widely contract specs use in the industry is Open API or Swagger documentation. This blog will introduce how to write contractor - Swagger can be in YAML or JSON.

  
 - Open API is the specification for machine-readable interface files
   for describing, producing, consuming, and visualizing RESTful web
   services.
   
 - Swagger is the Tools for implementing the specification. More details
   here: [https://swagger.io/](https://swagger.io/)

  

Swagger Introduction:

Let me introduce the Swagger tool, and I am showing the starting point. Feel free to be creative.

There are easy ways to start using Swagger tool:

1.  Online editor
2.  IDE like Visual Code

  But before that, let's dig some data.

  

# Data preparation


In the past - I used to work extensively with Climate data. So, I looked into earthquake data to use as a starting point for API. Here is the link for data definition: [https://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php](https://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php)

  
**Data Ingestion**

    import pandas as pd
    import feather
    import csv, requests

    csvurl ='http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_month.csv'
    rows = list(csv.DictReader(requests.get(csvurl).text.splitlines())) 
    df = pd.DataFrame(rows)

Simply use pandas and reading file in pandas data frame.

    #read data column 
    df.columns

There are multiple ways to store data. As data is not massive, we may save the data in a data.table or feather file.

    #move dataframe to feather file
    df.to_feather('output')

The output file is the feather file and will be our source of Truth! 

# Open API Specification

I will go with online editor - [https://editor.swagger.io/](https://editor.swagger.io/)


For our starting guide - We are building an API end-point for earthquake data we creating nd the intension is to pass the ID, which is earthquake ID and received information, which is in the data file.

<img src="../../assets/images/blackboard_series_1.PNG"  alt="High-level Overview" title="High-level Overview" width="350" height="350">


Below is the snapshot from the online editor and response. I have also hand-coded what those lines mean and how you can interpret them. It is the first example of a kick-start. Feel free to expand your knowledge by looking into the documentation page.


<img src="../../assets/images/API_1.png"  alt="Snapshot from Online Editor" title="Snapshot from Online Editor">
