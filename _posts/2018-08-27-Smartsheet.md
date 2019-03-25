---
bg: "smartsheet.png "
layout: post
date: 2018-08-27
mathjax: true
title: "Smartsheet Integration BigQuery"
crawlertitle: "Smartsheet Integration BigQuery"
categories: posts
tags: ['smartsheet', 'python', 'numpy', 'pandas']
author: Yash Karwa
jupyter: https://github.com/yashkarwa/
twitterImg: Preprocessing-for-deep-learning/rotation_small
excerpt: "The goal of the project is to scrape data from Smartsheet and Integration to BigQuery (Google Cloud Product) "
excerpt-image: '<img src="../../assets/images/smartsheet_bq.png" width="500" alt="Smartsheet Integration BigQuery" title="Smartsheet Integration BigQuery">
<em> snapshot</em>'
---

A notebook version of this post can be found [here](https://github.com/yashkarwa) on Github.

The goal of this post/notebook is to go from the basics of data preprocessing or scraping data from smartsheet, consolidate the data and store result in BigQuery.

Here is the table of contents of this tutorial:

1. **Load Packages**: In the first part, we will install smartsheet in python environment and load package in environment.

2. **Create Smartsheet ID's for each sheet**: In the second part, we will create Smartsheet Developer Key.

3. **Find Smartsheet ID's for each sheet**: In the third part, we will find smartsheet ID's for each sheet and extract data for identified sheet.

4. **Data Preprocessing**: In the fourth part, we will extract content of identified page.

5. **Load dataframe to BigQuery**: In the last part, load final dataframe to BigQuery.


## 1. Load Packages  


```python
import json
import os
import requests
import smartsheet, csv
from IPython.display import Image
import pandas.io.gbq as pd_gbq
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
```
