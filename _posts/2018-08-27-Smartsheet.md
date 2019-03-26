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

2. **Create Smartsheet Developer Key**: In the second part, we will create Smartsheet Developer Key.

3. **Find Smartsheet ID's for each sheet**: In the third part, we will find smartsheet ID's for each sheet and extract data for identified sheet.

4. **Data Preprocessing**: In the fourth part, we will extract content of identified page.

5. **Load dataframe to BigQuery**: In the last part, load final dataframe to BigQuery.


## 1. Install and Load Packages  


### Install command:

```python
pip install smartsheet-python-sdk

```
### Load packages :

```python
import json
import os
import requests
import smartsheet, csv
import pandas.io.gbq as pd_gbq
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
```

## 2. Create Smartsheet Developer Key 

A. Go to Smartsheet web portal > Profile > Apps & Integration
B. Click API Access > Generate new access token
C. Finally, make sure to save your API token.

**Note** : You cannot save your token later. In case missed, re-create a new one.


## 3. Find Smartsheet ID's for each sheet

```python
# Create base client object and set the access token
ss_client = smartsheet.Smartsheet('<INSERT KEY HERE>')

# Include all information 
response = ss_client.Sheets.list_sheets(include="attachments,source", include_all=True)
sheets = response.data

# print each sheet IDs and Name for further use (i.e. data extraction)
for single_sheet in response.data:
    print (single_sheet.id, single_sheet.name)
```

<pre class='output'>
123456789012  smartsheet_name_1
123456789013  smartsheet_name_2
</pre>