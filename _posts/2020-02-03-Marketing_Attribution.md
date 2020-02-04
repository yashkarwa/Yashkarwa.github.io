---
bg: "churn_analysis_2.jpg"
layout: post
date: 2020-02-03
mathjax: true
title: "Customer Churn"
crawlertitle: "Customer Churn"
categories: posts
tags: ['customer', 'data', 'marketing', 'attribution', 'heuristic', 'markov']
author: Yash Karwa
jupyter: https://github.com/yashkarwa/
twitterImg: 
excerpt: "The goal of the post is to share the practical application of Marketing Attribution"
excerpt-image: '<img src="../../assets/images/churn_analysis_2.jpg" width="250" alt="Churn Analysis" title="Churn Analysis">
<em> Customer Churn</em>'
---



### 1. Business Problem

We will be looking at Google Merchandise Store <https://shop.googlemerchandisestore.com/> Customer who loves Google Products can buy tees, jackets and others swags from this website. In the website, Google Analytics tracks every event on the website. They do have multiple tocuh-points like Youtube, Google Ads, Referaal links etc. 

Google has shared the data set (after normalization) in Google Big Query and we can use this data for **Channel Attribution**

------------

### 1.1 Problem Statement
There are multiple Conversion Model that are widely used in Marketing domains. Below are some of them: 

1. First Interaction
2. Last Interaction
3. Time Decay Interaction
4. Linear Interaction 

We can use any of the Heuristic Models and mostly used in the industry is the **Last Touch** Model. The problem in the heuristic model is not to consider all the different channels from the application, which will hurt the overall conversion funnel. 


![Source <https://github.com/MatCyt/Markov-Chain>](AttributionApproach.PNG)

The solution in this case study is to capture the whole journey by using **Markov Chain**. 

------

### 1.2 Data 

We are fetching the daata from Google BigQuery in public data set freely avaible. Below is the query to fetch data:



```r
# Databases
#library(bigrquery)
library(DBI)
#library(connections)
library(jsonlite)

# Core
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------------------------------------------------------------- tidyverse 1.3.0 --
```

```
## v ggplot2 3.2.1     v purrr   0.3.3
## v tibble  2.1.3     v dplyr   0.8.4
## v tidyr   1.0.2     v stringr 1.4.0
## v readr   1.3.1     v forcats 0.4.0
```

```
## -- Conflicts ------------------------------------------------------------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter()  masks stats::filter()
## x purrr::flatten() masks jsonlite::flatten()
## x dplyr::lag()     masks stats::lag()
```

```r
library(tidyquant)
```

```
## Loading required package: lubridate
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```
## Loading required package: PerformanceAnalytics
```

```
## Loading required package: xts
```

```
## Loading required package: zoo
```

```
## 
## Attaching package: 'zoo'
```

```
## The following objects are masked from 'package:base':
## 
##     as.Date, as.Date.numeric
```

```
## 
## Attaching package: 'xts'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     first, last
```

```
## 
## Attaching package: 'PerformanceAnalytics'
```

```
## The following object is masked from 'package:graphics':
## 
##     legend
```

```
## Loading required package: quantmod
```

```
## Loading required package: TTR
```

```
## Version 0.4-0 included new data defaults. See ?getSymbols.
```

```
## == Need to Learn tidyquant? ==========================================================================================================
## Business Science offers a 1-hour course - Learning Lab #9: Performance Analysis & Portfolio Optimization with tidyquant!
## </> Learn more at: https://university.business-science.io/p/learning-labs-pro </>
```

```r
library(lubridate)
library(plotly)
```

```
## 
## Attaching package: 'plotly'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     last_plot
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```

```
## The following object is masked from 'package:graphics':
## 
##     layout
```

```r
# dplyr helpers
library(dbplyr)
```

```
## 
## Attaching package: 'dbplyr'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     ident, sql
```

```r
# Attribution
library(ChannelAttribution)
```



```r
query <- "
SELECT fullVisitorId, date, channelGrouping,
    trafficSource.source AS traffic_source,
    SUM ( totals.transactions ) AS total_transactions, 
    SUM ( totals.totalTransactionRevenue ) AS total_transaction_revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
GROUP BY fullVisitorId, date, channelGrouping, traffic_source;
"
```

--------------------------------------------------


### 1.3 Data Cleaning 

In case, you don'thave bigQuery. I am sharing the bigQuery results in CSV file. 


```r
# Incase you don't need to run BigQuery; Use csv
query_tbl <- read_csv("data/bigquery_data.csv")
```

```
## Parsed with column specification:
## cols(
##   fullVisitorId = col_character(),
##   date = col_double(),
##   channelGrouping = col_character(),
##   traffic_source = col_character(),
##   total_transactions = col_double(),
##   total_transaction_revenue = col_double()
## )
```

```r
query_tbl %>%
    count(channelGrouping, traffic_source, sort = TRUE)
```

```
## # A tibble: 287 x 3
##    channelGrouping traffic_source            n
##    <chr>           <chr>                 <int>
##  1 Organic Search  google               212125
##  2 Social          youtube.com          209881
##  3 Organic Search  (direct)             137501
##  4 Direct          (direct)             131037
##  5 Referral        (direct)              61801
##  6 Affiliates      Partners              15062
##  7 Referral        analytics.google.com  14328
##  8 Paid Search     google                11592
##  9 Paid Search     (direct)              10913
## 10 Display         dfa                    4935
## # ... with 277 more rows
```

```r
query_tbl %>% count(channelGrouping, sort = TRUE)
```

```
## # A tibble: 8 x 2
##   channelGrouping      n
##   <chr>            <int>
## 1 Organic Search  356052
## 2 Social          222610
## 3 Direct          131037
## 4 Referral         94410
## 5 Paid Search      22506
## 6 Affiliates       15062
## 7 Display           5440
## 8 (Other)            107
```

```r
query_tbl %>% count(traffic_source, sort = TRUE)
```

```
## # A tibble: 275 x 2
##    traffic_source            n
##    <chr>                 <int>
##  1 (direct)             341327
##  2 google               224182
##  3 youtube.com          209881
##  4 Partners              15069
##  5 analytics.google.com  14328
##  6 dfa                    4935
##  7 google.com             4443
##  8 baidu                  3287
##  9 m.facebook.com         3275
## 10 sites.google.com       2735
## # ... with 265 more rows
```

```r
query_tbl %>% count(traffic_source, sort = TRUE) %>% glimpse()
```

```
## Observations: 275
## Variables: 2
## $ traffic_source <chr> "(direct)", "google", "youtube.com", "Partners"...
## $ n              <int> 341327, 224182, 209881, 15069, 14328, 4935, 444...
```

------------



```r
g <- query_tbl %>% 
    
    # Cleaning
    mutate(traffic_source_clean = traffic_source %>% 
               str_to_lower() %>%
               str_replace("\\.com", "") %>%
               str_replace("^m\\.", "")) %>%
    mutate(traffic_source_clean = case_when(
        str_detect(traffic_source_clean, "\\.google") ~ "google_product",
        str_detect(traffic_source_clean, "google\\.") ~ "google",
        str_detect(traffic_source_clean, "facebook") ~ "facebook",
        TRUE ~ traffic_source_clean
    )) %>%
    mutate(traffic_source_clean = traffic_source_clean %>%
               as_factor() %>%
               fct_lump(n = 3, other_level = "other_source")) %>%
    
    # Aggregating 
    count(traffic_source_clean) %>%
    arrange(desc(n)) %>% 
    mutate(traffic_source_clean = fct_reorder(traffic_source_clean, n)) %>%
    
    # Visualizing
    ggplot(aes(traffic_source_clean, n)) +
    geom_point(size = 5, color='darkblue') +
    coord_flip() +
    expand_limits(y = 0)



ggplotly(g)
```

<!--html_preserve--><div id="htmlwidget-e776a2ee5cb381a16447" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-e776a2ee5cb381a16447">{"x":{"data":[{"x":[341327,229515,209971,66411],"y":[4,3,2,1],"text":["traffic_source_clean: (direct)<br />n: 341327","traffic_source_clean: google<br />n: 229515","traffic_source_clean: youtube<br />n: 209971","traffic_source_clean: other_source<br />n:  66411"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(0,0,139,1)","opacity":1,"size":18.8976377952756,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(0,0,139,1)"}},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"visible":false,"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","x":[null],"y":[null],"frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":95.7077625570777},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-17066.35,358393.35],"tickmode":"array","ticktext":["0e+00","1e+05","2e+05","3e+05"],"tickvals":[0,100000,200000,300000],"categoryorder":"array","categoryarray":["0e+00","1e+05","2e+05","3e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":{"text":"n","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,4.6],"tickmode":"array","ticktext":["other_source","youtube","google","(direct)"],"tickvals":[1,2,3,4],"categoryorder":"array","categoryarray":["other_source","youtube","google","(direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":{"text":"traffic_source_clean","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"1088567e1031":{"x":{},"y":{},"type":"scatter"},"108815a34dcf":{"y":{}}},"cur_data":"1088567e1031","visdat":{"1088567e1031":["function (y) ","x"],"108815a34dcf":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

I have defined into four categories : Direct , Google , Youtube and others (all other sources are added in this group.)

---------------


```r
# Apply Cleaning Steps
query_clean_tbl <- query_tbl %>% 
    mutate(traffic_source_clean = traffic_source %>% 
               str_to_lower() %>%
               str_replace("\\.com", "") %>%
               str_replace("^m\\.", "")) %>%
    mutate(traffic_source_clean = case_when(
        str_detect(traffic_source_clean, "\\.google") ~ "google_product",
        str_detect(traffic_source_clean, "google\\.") ~ "google",
        str_detect(traffic_source_clean, "facebook") ~ "facebook",
        TRUE ~ traffic_source_clean
    )) %>%
    mutate(traffic_source_clean = traffic_source_clean %>%
               as_factor() %>%
               fct_lump(n = 3, other_level = "other_source"))
    
query_clean_tbl %>% count(channelGrouping, traffic_source_clean, sort = TRUE)
```

```
## # A tibble: 19 x 3
##    channelGrouping traffic_source_clean      n
##    <chr>           <fct>                 <int>
##  1 Organic Search  google               212125
##  2 Social          youtube              209971
##  3 Organic Search  (direct)             137501
##  4 Direct          (direct)             131037
##  5 Referral        (direct)              61801
##  6 Referral        other_source          27276
##  7 Affiliates      other_source          15062
##  8 Social          other_source          12639
##  9 Paid Search     google                11592
## 10 Paid Search     (direct)              10913
## 11 Organic Search  other_source           6426
## 12 Referral        google                 5333
## 13 Display         other_source           4935
## 14 Display         google                  432
## 15 Display         (direct)                 73
## 16 (Other)         other_source             72
## 17 (Other)         google                   33
## 18 (Other)         (direct)                  2
## 19 Paid Search     other_source              1
```

-------------------

## 2. Understand Value by Channel-Source - Attribution Analysis


```r
# Attribution Analysis
channel_value_tbl <- query_clean_tbl %>%
    group_by(channelGrouping, traffic_source_clean) %>%
    summarize(
        n = n(),
        total_transactions = sum(as.numeric(total_transactions), na.rm = TRUE),
        total_transaction_revenue = sum(as.numeric(total_transaction_revenue), na.rm = TRUE) / 1e6
    ) %>%
    mutate(revenue_per_conversion = total_transaction_revenue / n) %>%
    ungroup() %>%
    arrange(desc(total_transaction_revenue))

channel_value_tbl
```

```
## # A tibble: 19 x 6
##    channelGrouping traffic_source_~      n total_transacti~
##    <chr>           <fct>             <int>            <dbl>
##  1 Referral        (direct)          61801             5348
##  2 Direct          (direct)         131037             2219
##  3 Organic Search  google           212125             2174
##  4 Organic Search  (direct)         137501             1360
##  5 Referral        other_source      27276              190
##  6 Paid Search     google            11592              248
##  7 Display         other_source       4935              132
##  8 Paid Search     (direct)          10913              231
##  9 Social          other_source      12639              120
## 10 Organic Search  other_source       6426               47
## 11 Display         google              432               18
## 12 Affiliates      other_source      15062                9
## 13 Referral        google             5333                5
## 14 Social          youtube          209971               11
## 15 Display         (direct)             73                2
## 16 (Other)         google               33                1
## 17 (Other)         (direct)              2                0
## 18 (Other)         other_source         72                0
## 19 Paid Search     other_source          1                0
## # ... with 2 more variables: total_transaction_revenue <dbl>,
## #   revenue_per_conversion <dbl>
```


----

## 3. Data Visulization 


### 3.1 Visualize Average Revenue Per Channel


```r
#Visualize Average Revenue Per Channel
g <- channel_value_tbl %>%
    
    # Make plot labels and sort order
    mutate(point_text = str_glue("Total Revenue: {scales::dollar(total_transaction_revenue)}
                                 Revenue Per Entry: {scales::dollar(revenue_per_conversion)}")) %>%
    mutate(channel_source = str_glue("{channelGrouping} - {traffic_source_clean}")) %>%
    mutate(channel_source = as_factor(channel_source) %>% fct_reorder(revenue_per_conversion)) %>%
    
    # Visualize
    ggplot(aes(channel_source, revenue_per_conversion)) +
    geom_point(aes(text = point_text, color = total_transaction_revenue), size = 5) +
    coord_flip() +
    scale_y_continuous(labels = scales::dollar_format()) +
    scale_color_continuous(labels = scales::dollar_format()) +
    labs(title = "Average Revenue Per Channel Channel", color = "Revenue") +
    theme_dark()
```

```
## Warning: Ignoring unknown aesthetics: text
```

```r
ggplotly(g, tooltip = "text")
```

<!--html_preserve--><div id="htmlwidget-7a78dbfaf7b24b65e5fd" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-7a78dbfaf7b24b65e5fd">{"x":{"data":[{"x":[9.92487192763871,2.88069194197059,0.990874390100177,0.851809295932393,1.31994317348585,2.40478347135956,3.99507396149949,1.80217905250619,0.636261571326845,0.409442888266418,3.40773148148148,0.0434457575355199,0.0712994562160135,0.00169104304880198,1.05849315068493,0.363333333333333,0,0,0],"y":[19,16,11,10,13,15,18,14,9,8,17,5,6,4,12,7,1,2,3],"text":["Total Revenue: $613,367<br />Revenue Per Entry: $9.92","Total Revenue: $377,477<br />Revenue Per Entry: $2.88","Total Revenue: $210,189<br />Revenue Per Entry: $0.99","Total Revenue: $117,125<br />Revenue Per Entry: $0.85","Total Revenue: $36,003<br />Revenue Per Entry: $1.32","Total Revenue: $27,876<br />Revenue Per Entry: $2.40","Total Revenue: $19,716<br />Revenue Per Entry: $4.00","Total Revenue: $19,667<br />Revenue Per Entry: $1.80","Total Revenue: $8,042<br />Revenue Per Entry: $0.64","Total Revenue: $2,631<br />Revenue Per Entry: $0.41","Total Revenue: $1,472<br />Revenue Per Entry: $3.41","Total Revenue: $654<br />Revenue Per Entry: $0.04","Total Revenue: $380<br />Revenue Per Entry: $0.07","Total Revenue: $355<br />Revenue Per Entry: $0.00","Total Revenue: $77<br />Revenue Per Entry: $1.06","Total Revenue: $12<br />Revenue Per Entry: $0.36","Total Revenue: $0<br />Revenue Per Entry: $0.00","Total Revenue: $0<br />Revenue Per Entry: $0.00","Total Revenue: $0<br />Revenue Per Entry: $0.00"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":["rgba(86,177,247,1)","rgba(59,122,173,1)","rgba(40,85,124,1)","rgba(31,66,98,1)","rgba(23,50,76,1)","rgba(22,48,74,1)","rgba(21,47,72,1)","rgba(21,47,72,1)","rgba(20,45,69,1)","rgba(19,43,68,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)"],"opacity":1,"size":18.8976377952756,"symbol":"circle","line":{"width":1.88976377952756,"color":["rgba(86,177,247,1)","rgba(59,122,173,1)","rgba(40,85,124,1)","rgba(31,66,98,1)","rgba(23,50,76,1)","rgba(22,48,74,1)","rgba(21,47,72,1)","rgba(21,47,72,1)","rgba(20,45,69,1)","rgba(19,43,68,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)"]}},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1],"y":[0],"name":"99_d014dd8e6a1c041072a3ac76770b4b04","type":"scatter","mode":"markers","opacity":0,"hoverinfo":"skip","showlegend":false,"marker":{"color":[0,1],"colorscale":[[0,"#132B43"],[0.0526315789473684,"#16314B"],[0.105263157894737,"#193754"],[0.157894736842105,"#1D3E5C"],[0.210526315789474,"#204465"],[0.263157894736842,"#234B6E"],[0.315789473684211,"#275277"],[0.368421052631579,"#2A5980"],[0.421052631578947,"#2E608A"],[0.473684210526316,"#316793"],[0.526315789473684,"#356E9D"],[0.578947368421053,"#3875A6"],[0.631578947368421,"#3C7CB0"],[0.684210526315789,"#3F83BA"],[0.736842105263158,"#438BC4"],[0.789473684210526,"#4792CE"],[0.842105263157895,"#4B9AD8"],[0.894736842105263,"#4EA2E2"],[0.947368421052632,"#52A9ED"],[1,"#56B1F7"]],"colorbar":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"thickness":23.04,"title":"Revenue","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"tickmode":"array","ticktext":["$0","$200,000","$400,000","$600,000"],"tickvals":[0,0.326069052849777,0.652138105699555,0.978207158549332],"tickfont":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"ticklen":2,"len":0.5}},"xaxis":"x","yaxis":"y","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":195.068493150685},"plot_bgcolor":"rgba(127,127,127,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"Average Revenue Per Channel Channel","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-0.496243596381936,10.4211155240206],"tickmode":"array","ticktext":["$0.00","$2.50","$5.00","$7.50","$10.00"],"tickvals":[0,2.5,5,7.5,10],"categoryorder":"array","categoryarray":["$0.00","$2.50","$5.00","$7.50","$10.00"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(107,107,107,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"y","title":{"text":"revenue_per_conversion","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other) - (direct)","(Other) - other_source","Paid Search - other_source","Social - youtube","Affiliates - other_source","Referral - google","(Other) - google","Organic Search - other_source","Social - other_source","Organic Search - (direct)","Organic Search - google","Display - (direct)","Referral - other_source","Paid Search - (direct)","Paid Search - google","Direct - (direct)","Display - google","Display - other_source","Referral - (direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other) - (direct)","(Other) - other_source","Paid Search - other_source","Social - youtube","Affiliates - other_source","Referral - google","(Other) - google","Organic Search - other_source","Social - other_source","Organic Search - (direct)","Organic Search - google","Display - (direct)","Referral - other_source","Paid Search - (direct)","Paid Search - google","Direct - (direct)","Display - google","Display - other_source","Referral - (direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(107,107,107,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"x","title":{"text":"channel_source","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"1088f7e1c73":{"text":{},"colour":{},"x":{},"y":{},"type":"scatter"}},"cur_data":"1088f7e1c73","visdat":{"1088f7e1c73":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

------------------



### 3.2 Visualize  Total Revenue Per Channel


```r
g <- channel_value_tbl %>%
    
    # Make plot labels and sort order
    mutate(point_text = str_glue("Total Revenue: {scales::dollar(total_transaction_revenue)}
                                 Revenue Per Entry: {scales::dollar(revenue_per_conversion)}")) %>%
    mutate(channel_source = str_glue("{channelGrouping} - {traffic_source_clean}")) %>%
    mutate(channel_source = as_factor(channel_source) %>% fct_reorder(total_transaction_revenue)) %>%
    
    # Visualize
    ggplot(aes(channel_source, total_transaction_revenue)) +
    geom_point(aes(text = point_text, color = revenue_per_conversion), size = 5) +
    coord_flip() +
    scale_y_continuous(labels = scales::dollar_format()) +
    scale_color_continuous(labels = scales::dollar_format()) +
    labs(title = "Total Revenue Per Channel Channel", color = "RPE") +
    theme_dark()
```

```
## Warning: Ignoring unknown aesthetics: text
```

```r
ggplotly(g, tooltip = "text")
```

<!--html_preserve--><div id="htmlwidget-b03ab5e0cf4c450aab4b" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-b03ab5e0cf4c450aab4b">{"x":{"data":[{"x":[613367.01,377477.23,210189.23,117124.63,36002.77,27876.25,19715.69,19667.18,8041.71,2631.08,1472.14,654.38,380.24,355.07,77.27,11.99,0,0,0],"y":[19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,1,2,3],"text":["Total Revenue: $613,367<br />Revenue Per Entry: $9.92","Total Revenue: $377,477<br />Revenue Per Entry: $2.88","Total Revenue: $210,189<br />Revenue Per Entry: $0.99","Total Revenue: $117,125<br />Revenue Per Entry: $0.85","Total Revenue: $36,003<br />Revenue Per Entry: $1.32","Total Revenue: $27,876<br />Revenue Per Entry: $2.40","Total Revenue: $19,716<br />Revenue Per Entry: $4.00","Total Revenue: $19,667<br />Revenue Per Entry: $1.80","Total Revenue: $8,042<br />Revenue Per Entry: $0.64","Total Revenue: $2,631<br />Revenue Per Entry: $0.41","Total Revenue: $1,472<br />Revenue Per Entry: $3.41","Total Revenue: $654<br />Revenue Per Entry: $0.04","Total Revenue: $380<br />Revenue Per Entry: $0.07","Total Revenue: $355<br />Revenue Per Entry: $0.00","Total Revenue: $77<br />Revenue Per Entry: $1.06","Total Revenue: $12<br />Revenue Per Entry: $0.36","Total Revenue: $0<br />Revenue Per Entry: $0.00","Total Revenue: $0<br />Revenue Per Entry: $0.00","Total Revenue: $0<br />Revenue Per Entry: $0.00"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":["rgba(86,177,247,1)","rgba(37,79,115,1)","rgba(25,55,83,1)","rgba(24,53,81,1)","rgba(27,59,88,1)","rgba(34,72,107,1)","rgba(44,93,134,1)","rgba(30,65,96,1)","rgba(23,51,77,1)","rgba(21,48,74,1)","rgba(40,85,124,1)","rgba(19,44,68,1)","rgba(19,44,68,1)","rgba(19,43,67,1)","rgba(25,56,84,1)","rgba(21,47,73,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)"],"opacity":1,"size":18.8976377952756,"symbol":"circle","line":{"width":1.88976377952756,"color":["rgba(86,177,247,1)","rgba(37,79,115,1)","rgba(25,55,83,1)","rgba(24,53,81,1)","rgba(27,59,88,1)","rgba(34,72,107,1)","rgba(44,93,134,1)","rgba(30,65,96,1)","rgba(23,51,77,1)","rgba(21,48,74,1)","rgba(40,85,124,1)","rgba(19,44,68,1)","rgba(19,44,68,1)","rgba(19,43,67,1)","rgba(25,56,84,1)","rgba(21,47,73,1)","rgba(19,43,67,1)","rgba(19,43,67,1)","rgba(19,43,67,1)"]}},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1],"y":[0],"name":"99_036bd1f794e7aa2ef90bf52d46f89cd8","type":"scatter","mode":"markers","opacity":0,"hoverinfo":"skip","showlegend":false,"marker":{"color":[0,1],"colorscale":[[0,"#132B43"],[0.0526315789473684,"#16314B"],[0.105263157894737,"#193754"],[0.157894736842105,"#1D3E5C"],[0.210526315789474,"#204465"],[0.263157894736842,"#234B6E"],[0.315789473684211,"#275277"],[0.368421052631579,"#2A5980"],[0.421052631578947,"#2E608A"],[0.473684210526316,"#316793"],[0.526315789473684,"#356E9D"],[0.578947368421053,"#3875A6"],[0.631578947368421,"#3C7CB0"],[0.684210526315789,"#3F83BA"],[0.736842105263158,"#438BC4"],[0.789473684210526,"#4792CE"],[0.842105263157895,"#4B9AD8"],[0.894736842105263,"#4EA2E2"],[0.947368421052631,"#52A9ED"],[1,"#56B1F7"]],"colorbar":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"thickness":23.04,"title":"RPE","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"tickmode":"array","ticktext":["$0.00","$2.50","$5.00","$7.50"],"tickvals":[0,0.251892419189614,0.503784838379227,0.755677257568841],"tickfont":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"ticklen":2,"len":0.5}},"xaxis":"x","yaxis":"y","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":195.068493150685},"plot_bgcolor":"rgba(127,127,127,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"Total Revenue Per Channel Channel","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-30668.3505,644035.3605],"tickmode":"array","ticktext":["$0","$200,000","$400,000","$600,000"],"tickvals":[0,200000,400000,600000],"categoryorder":"array","categoryarray":["$0","$200,000","$400,000","$600,000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(107,107,107,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"y","title":{"text":"total_transaction_revenue","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other) - (direct)","(Other) - other_source","Paid Search - other_source","(Other) - google","Display - (direct)","Social - youtube","Referral - google","Affiliates - other_source","Display - google","Organic Search - other_source","Social - other_source","Paid Search - (direct)","Display - other_source","Paid Search - google","Referral - other_source","Organic Search - (direct)","Organic Search - google","Direct - (direct)","Referral - (direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other) - (direct)","(Other) - other_source","Paid Search - other_source","(Other) - google","Display - (direct)","Social - youtube","Referral - google","Affiliates - other_source","Display - google","Organic Search - other_source","Social - other_source","Paid Search - (direct)","Display - other_source","Paid Search - google","Referral - other_source","Organic Search - (direct)","Organic Search - google","Direct - (direct)","Referral - (direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.33208800332088,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(107,107,107,1)","gridwidth":0.33208800332088,"zeroline":false,"anchor":"x","title":{"text":"channel_source","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"108857ffad":{"text":{},"colour":{},"x":{},"y":{},"type":"scatter"}},"cur_data":"108857ffad","visdat":{"108857ffad":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


-------------------

# 4. Data volume is in ~700K rows and used dtplyr (big data for data table)


```r
query_dtplyr <- query_clean_tbl %>% dtplyr::lazy_dt()


visitor_date_channelgrouping_dtplyr <- query_dtplyr %>%
    
    # Fix date from BigQuery
    mutate(date = ymd(date)) %>%
    
    # Add Channel-Traffic Source Combo
    mutate(channel_source = str_c(channelGrouping, "-", traffic_source_clean)) %>%
    
    # Select Important columns
    select(fullVisitorId, date, channel_source, total_transactions, total_transaction_revenue) %>%
    
    # Fix revenue
    mutate(total_transaction_revenue = total_transaction_revenue / 1e6) %>%
    
    # Path Calculations
    group_by(fullVisitorId, date, channel_source) %>%
    mutate(total_transactions = sum(total_transactions, na.rm = TRUE)) %>%
    mutate(total_transaction_revenue = sum(total_transaction_revenue, na.rm = TRUE)) %>%
    arrange(date) %>%
    ungroup() %>%
    
    # Binary Conversion (Yes/No)
    mutate(total_transactions = ifelse(total_transactions > 0, 1, 0))

visitor_date_channelgrouping_tbl <- as_tibble(visitor_date_channelgrouping_dtplyr)
visitor_date_channelgrouping_tbl
```

```
## # A tibble: 847,224 x 5
##    fullVisitorId date       channel_source total_transacti~
##    <chr>         <date>     <chr>                     <dbl>
##  1 000722514342~ 2016-08-01 Direct-(direc~                0
##  2 001465993518~ 2016-08-01 Social-youtube                0
##  3 001569443280~ 2016-08-01 Direct-(direc~                0
##  4 001747219368~ 2016-08-01 Organic Searc~                0
##  5 001852659575~ 2016-08-01 Social-youtube                0
##  6 001998304603~ 2016-08-01 Direct-(direc~                0
##  7 002597793543~ 2016-08-01 Social-youtube                0
##  8 002855959112~ 2016-08-01 Direct-(direc~                0
##  9 003065339361~ 2016-08-01 Social-youtube                0
## 10 003651776577~ 2016-08-01 Social-youtube                0
## # ... with 847,214 more rows, and 1 more variable:
## #   total_transaction_revenue <dbl>
```



```r
channel_path_dtplyr <- visitor_date_channelgrouping_dtplyr %>%
    
    group_by(fullVisitorId) %>%
    summarize(
        channel_path     = str_c(channel_source, collapse = " > "),
        conversion_total = sum(total_transactions),
        conversion_null  = sum(total_transactions == 0),
        conversion_value = sum(total_transaction_revenue),
        n_channel_path   = n()
    ) %>%
    ungroup() 

channel_path_tbl <- channel_path_dtplyr %>% as_tibble()

# Longest Channel Path?
channel_path_tbl %>% arrange(desc(n_channel_path))
```

```
## # A tibble: 714,167 x 6
##    fullVisitorId channel_path conversion_total conversion_null
##    <chr>         <chr>                   <dbl>           <int>
##  1 195745897629~ Organic Sea~               14             132
##  2 360847519334~ Paid Search~                1             130
##  3 072031119776~ Social-othe~                0             119
##  4 082483972611~ Organic Sea~                0             117
##  5 403807668303~ Organic Sea~                0             110
##  6 185674914791~ Organic Sea~                0             104
##  7 326983486538~ Direct-(dir~                1              87
##  8 601877531773~ Direct-(dir~                0              83
##  9 980127621496~ Organic Sea~                0              83
## 10 369423402852~ Direct-(dir~                1              73
## # ... with 714,157 more rows, and 2 more variables:
## #   conversion_value <dbl>, n_channel_path <int>
```

```r
# Most Path Conversions?
channel_path_tbl %>% arrange(desc(conversion_total))
```

```
## # A tibble: 714,167 x 6
##    fullVisitorId channel_path conversion_total conversion_null
##    <chr>         <chr>                   <dbl>           <int>
##  1 781314996140~ Direct-(dir~               28              22
##  2 498436650112~ Direct-(dir~               16               5
##  3 240252719973~ Direct-(dir~               15              13
##  4 195745897629~ Organic Sea~               14             132
##  5 676073240225~ Referral-(d~               14              15
##  6 060891519773~ Referral-(d~               13               2
##  7 731124288608~ Direct-(dir~               12              10
##  8 207416433864~ Direct-(dir~               11               8
##  9 819787964379~ Organic Sea~               10              27
## 10 908913239224~ Direct-(dir~               10               9
## # ... with 714,157 more rows, and 2 more variables:
## #   conversion_value <dbl>, n_channel_path <int>
```


-----------

# 5. CHANNEL ATTRIBUTION MODELING


## 5.1 Plotting Utility 

```r
plot_attribution <- function(data, title = "Attribution Model", interactive = TRUE) {
    g <- data %>%
        pivot_longer(
            cols = -channel_name,
            names_to  = "conversion_type",
            values_to = "conversion_value"
        ) %>%
        mutate(channel_name = as_factor(channel_name) %>% fct_reorder(conversion_value)) %>%
        
        ggplot(aes(channel_name, conversion_value, fill = conversion_type)) +
        geom_col(position = "dodge") +
        theme_tq() +
        scale_fill_tq() +
        coord_flip() +
        labs(title = title)
    
    if (interactive) return(ggplotly(g))
    else return(g)
}
```

--------

## 5.2 Heurisitic Models 


```r
channel_path_heuristic_model <- channel_path_tbl %>%
    heuristic_models(
        var_path = "channel_path",
        var_conv = "conversion_total",
        var_value = "conversion_value",
        sep = ">"
    )
```

-----------

## 5.3 Data Visualization


```r
# Conversion Count
channel_path_heuristic_model %>%
    select_at(vars(channel_name, matches("conversions"))) %>%
    plot_attribution()
```

<!--html_preserve--><div id="htmlwidget-dc6295894b0bab4b1eb7" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-dc6295894b0bab4b1eb7">{"x":{"data":[{"orientation":"h","width":[0.3,0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999999,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,1,2,5,12,14,21,42,67,138,131,218,216,1370,2132,2662,4429],"y":[0.7,1.7,2.7,3.7,4.7,5.7,6.7,7.7,8.7,9.7,10.7,11.7,12.7,13.7,14.7,15.7,16.7,17.7,18.7],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: (Other)-google<br />conversion_value:    1.000000<br />conversion_type: first_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    2.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.000000<br />conversion_type: first_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:   12.000000<br />conversion_type: first_touch_conversions","channel_name: Social-youtube<br />conversion_value:   14.000000<br />conversion_type: first_touch_conversions","channel_name: Display-google<br />conversion_value:   21.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   42.000000<br />conversion_type: first_touch_conversions","channel_name: Social-other_source<br />conversion_value:   67.000000<br />conversion_type: first_touch_conversions","channel_name: Display-other_source<br />conversion_value:  138.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  131.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  218.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  216.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1370.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2132.000000<br />conversion_type: first_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 2662.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 4429.000000<br />conversion_type: first_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(44,62,80,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"first_touch_conversions","legendgroup":"first_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.3,0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999999,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,2,1,5,9,12,20,44,96,144,225,248,305,1283,2050,1893,5123],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: (Other)-google<br />conversion_value:    2.000000<br />conversion_type: last_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    1.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.000000<br />conversion_type: last_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:    9.000000<br />conversion_type: last_touch_conversions","channel_name: Social-youtube<br />conversion_value:   12.000000<br />conversion_type: last_touch_conversions","channel_name: Display-google<br />conversion_value:   20.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   44.000000<br />conversion_type: last_touch_conversions","channel_name: Social-other_source<br />conversion_value:   96.000000<br />conversion_type: last_touch_conversions","channel_name: Display-other_source<br />conversion_value:  144.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  225.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  248.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  305.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1283.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2050.000000<br />conversion_type: last_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 1893.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 5123.000000<br />conversion_type: last_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(227,26,28,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"last_touch_conversions","legendgroup":"last_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999997,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0.5,0,1.51515151515152,1.83333333333333,5.01351351351351,10.4353603603604,13.4861281736282,19.2801587301587,45.0039682539683,83.5416274470198,148.451147977188,180.76628462624,237.295185090705,258.901602850109,1331.86985217911,2088.38329821075,2186.5327851989,4847.19060253986],"y":[1.3,2.3,3.3,4.3,5.3,6.3,7.3,8.3,9.3,10.3,11.3,12.3,13.3,14.3,15.3,16.3,17.3,18.3,19.3],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: linear_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.500000<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: linear_touch_conversions","channel_name: (Other)-google<br />conversion_value:    1.515152<br />conversion_type: linear_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    1.833333<br />conversion_type: linear_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.013514<br />conversion_type: linear_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:   10.435360<br />conversion_type: linear_touch_conversions","channel_name: Social-youtube<br />conversion_value:   13.486128<br />conversion_type: linear_touch_conversions","channel_name: Display-google<br />conversion_value:   19.280159<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   45.003968<br />conversion_type: linear_touch_conversions","channel_name: Social-other_source<br />conversion_value:   83.541627<br />conversion_type: linear_touch_conversions","channel_name: Display-other_source<br />conversion_value:  148.451148<br />conversion_type: linear_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  180.766285<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  237.295185<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  258.901603<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1331.869852<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2088.383298<br />conversion_type: linear_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 2186.532785<br />conversion_type: linear_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 4847.190603<br />conversion_type: linear_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(24,188,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"linear_touch_conversions","legendgroup":"linear_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":183.37899543379},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Attribution Model","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-256.15,5379.15],"tickmode":"array","ticktext":["0","1000","2000","3000","4000","5000"],"tickvals":[0,1000,2000,3000,4000,5000],"categoryorder":"array","categoryarray":["0","1000","2000","3000","4000","5000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"y","title":{"text":"conversion_value","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other)-(direct)","(Other)-other_source","Paid Search-other_source","(Other)-google","Display-(direct)","Referral-google","Affiliates-other_source","Social-youtube","Display-google","Organic Search-other_source","Social-other_source","Display-other_source","Referral-other_source","Paid Search-(direct)","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other)-(direct)","(Other)-other_source","Paid Search-other_source","(Other)-google","Display-(direct)","Referral-google","Affiliates-other_source","Social-youtube","Display-google","Organic Search-other_source","Social-other_source","Display-other_source","Referral-other_source","Paid Search-(direct)","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"x","title":{"text":"channel_name","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0.33208800332088,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"conversion_type","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"10882eb624f0":{"x":{},"y":{},"fill":{},"type":"bar"}},"cur_data":"10882eb624f0","visdat":{"10882eb624f0":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->



```r
# Conversion Value
channel_path_heuristic_model %>%
    select_at(vars(channel_name, matches("value"))) %>%
    plot_attribution()
```

<!--html_preserve--><div id="htmlwidget-c2469517c43e42c5b16e" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-c2469517c43e42c5b16e">{"x":{"data":[{"orientation":"h","width":[0.3,0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999999,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,50.61,11.99,455.22,442.53,753.61,1955.17,2258.37,5624.51,17192.57,18199.92,25047.94,27580.11,125247.940000001,214005.6,476648.579999998,520184.050000001],"y":[0.7,1.7,2.7,3.7,4.7,5.7,6.7,7.7,8.7,9.7,10.7,11.7,12.7,13.7,14.7,15.7,16.7,17.7,18.7],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: (Other)-other_source<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: Display-(direct)<br />conversion_value:     50.61000<br />conversion_type: first_touch_value","channel_name: (Other)-google<br />conversion_value:     11.99000<br />conversion_type: first_touch_value","channel_name: Referral-google<br />conversion_value:    455.22000<br />conversion_type: first_touch_value","channel_name: Social-youtube<br />conversion_value:    442.53000<br />conversion_type: first_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    753.61000<br />conversion_type: first_touch_value","channel_name: Display-google<br />conversion_value:   1955.17000<br />conversion_type: first_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2258.37000<br />conversion_type: first_touch_value","channel_name: Social-other_source<br />conversion_value:   5624.51000<br />conversion_type: first_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  17192.57000<br />conversion_type: first_touch_value","channel_name: Display-other_source<br />conversion_value:  18199.92000<br />conversion_type: first_touch_value","channel_name: Paid Search-google<br />conversion_value:  25047.94000<br />conversion_type: first_touch_value","channel_name: Referral-other_source<br />conversion_value:  27580.11000<br />conversion_type: first_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 125247.94000<br />conversion_type: first_touch_value","channel_name: Organic Search-google<br />conversion_value: 214005.60000<br />conversion_type: first_touch_value","channel_name: Direct-(direct)<br />conversion_value: 476648.58000<br />conversion_type: first_touch_value","channel_name: Referral-(direct)<br />conversion_value: 520184.05000<br />conversion_type: first_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(44,62,80,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"first_touch_value","legendgroup":"first_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.3,0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999999,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,36.98,133.56,380.24,503.44,719.13,2426.64,2443.44,8456.99,21331.18,23233.75,51407.14,32291.44,115490.880000001,210736.94,335691.439999999,630375.529999995],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: (Other)-other_source<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: Display-(direct)<br />conversion_value:     36.98000<br />conversion_type: last_touch_value","channel_name: (Other)-google<br />conversion_value:    133.56000<br />conversion_type: last_touch_value","channel_name: Referral-google<br />conversion_value:    380.24000<br />conversion_type: last_touch_value","channel_name: Social-youtube<br />conversion_value:    503.44000<br />conversion_type: last_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    719.13000<br />conversion_type: last_touch_value","channel_name: Display-google<br />conversion_value:   2426.64000<br />conversion_type: last_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2443.44000<br />conversion_type: last_touch_value","channel_name: Social-other_source<br />conversion_value:   8456.99000<br />conversion_type: last_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  21331.18000<br />conversion_type: last_touch_value","channel_name: Display-other_source<br />conversion_value:  23233.75000<br />conversion_type: last_touch_value","channel_name: Paid Search-google<br />conversion_value:  51407.14000<br />conversion_type: last_touch_value","channel_name: Referral-other_source<br />conversion_value:  32291.44000<br />conversion_type: last_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 115490.88000<br />conversion_type: last_touch_value","channel_name: Organic Search-google<br />conversion_value: 210736.94000<br />conversion_type: last_touch_value","channel_name: Direct-(direct)<br />conversion_value: 335691.44000<br />conversion_type: last_touch_value","channel_name: Referral-(direct)<br />conversion_value: 630375.53000<br />conversion_type: last_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(227,26,28,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"last_touch_value","legendgroup":"last_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.3,0.3,0.3,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.300000000000001,0.299999999999997,0.299999999999997,0.299999999999997,0.299999999999997],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,24.43125,0,57.225,58.1369696969697,418.016351351351,487.855980241605,739.025952702703,2078.98149206349,2614.68153174603,7544.37124714773,22960.7187020122,21841.8934948129,31427.8168779343,33039.408699502,121494.924251787,210524.328621619,392847.101720134,587499.801857245],"y":[1.3,2.3,3.3,4.3,5.3,6.3,7.3,8.3,9.3,10.3,11.3,12.3,13.3,14.3,15.3,16.3,17.3,18.3,19.3],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: linear_touch_value","channel_name: (Other)-other_source<br />conversion_value:     24.43125<br />conversion_type: linear_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: linear_touch_value","channel_name: Display-(direct)<br />conversion_value:     57.22500<br />conversion_type: linear_touch_value","channel_name: (Other)-google<br />conversion_value:     58.13697<br />conversion_type: linear_touch_value","channel_name: Referral-google<br />conversion_value:    418.01635<br />conversion_type: linear_touch_value","channel_name: Social-youtube<br />conversion_value:    487.85598<br />conversion_type: linear_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    739.02595<br />conversion_type: linear_touch_value","channel_name: Display-google<br />conversion_value:   2078.98149<br />conversion_type: linear_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2614.68153<br />conversion_type: linear_touch_value","channel_name: Social-other_source<br />conversion_value:   7544.37125<br />conversion_type: linear_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  22960.71870<br />conversion_type: linear_touch_value","channel_name: Display-other_source<br />conversion_value:  21841.89349<br />conversion_type: linear_touch_value","channel_name: Paid Search-google<br />conversion_value:  31427.81688<br />conversion_type: linear_touch_value","channel_name: Referral-other_source<br />conversion_value:  33039.40870<br />conversion_type: linear_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 121494.92425<br />conversion_type: linear_touch_value","channel_name: Organic Search-google<br />conversion_value: 210524.32862<br />conversion_type: linear_touch_value","channel_name: Direct-(direct)<br />conversion_value: 392847.10172<br />conversion_type: linear_touch_value","channel_name: Referral-(direct)<br />conversion_value: 587499.80186<br />conversion_type: linear_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(24,188,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"linear_touch_value","legendgroup":"linear_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":183.37899543379},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Attribution Model","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-31518.7764999998,661894.306499995],"tickmode":"array","ticktext":["0e+00","2e+05","4e+05","6e+05"],"tickvals":[0,200000,400000,600000],"categoryorder":"array","categoryarray":["0e+00","2e+05","4e+05","6e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"y","title":{"text":"conversion_value","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other)-(direct)","(Other)-other_source","Paid Search-other_source","Display-(direct)","(Other)-google","Referral-google","Social-youtube","Affiliates-other_source","Display-google","Organic Search-other_source","Social-other_source","Paid Search-(direct)","Display-other_source","Paid Search-google","Referral-other_source","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other)-(direct)","(Other)-other_source","Paid Search-other_source","Display-(direct)","(Other)-google","Referral-google","Social-youtube","Affiliates-other_source","Display-google","Organic Search-other_source","Social-other_source","Paid Search-(direct)","Display-other_source","Paid Search-google","Referral-other_source","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"x","title":{"text":"channel_name","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0.33208800332088,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"conversion_type","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"108821f837f1":{"x":{},"y":{},"fill":{},"type":"bar"}},"cur_data":"108821f837f1","visdat":{"108821f837f1":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

--------------------

## 6 Markov Chain


```r
markov_model <- channel_path_tbl %>%
    markov_model_mp(
        var_path = "channel_path",
        var_conv = "conversion_total",
        var_value = "conversion_value",
        sep = ">", 
        order = 3, 
        nfold = 10,
        seed = 123
    )
```

```
## Number of simulations: 100000 - Convergence reached: 0.028860
```

```r
# Conversion Count
markov_model %>%
    select(-total_conversion_value) %>%
    rename(markov = total_conversion) %>%
    left_join(
        channel_path_heuristic_model %>%
            select_at(vars(channel_name, matches("conversions")))
    ) %>%
    plot_attribution()
```

```
## Joining, by = "channel_name"
```

<!--html_preserve--><div id="htmlwidget-8bde122cde25ffb56374" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-8bde122cde25ffb56374">{"x":{"data":[{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,1,2,5,12,14,21,42,67,138,131,218,216,1370,2132,2662,4429],"y":[0.662500000000001,1.6625,2.6625,3.6625,4.6625,5.6625,6.6625,7.6625,8.6625,9.6625,10.6625,11.6625,12.6625,13.6625,14.6625,15.6625,16.6625,17.6625,18.6625],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.000000<br />conversion_type: first_touch_conversions","channel_name: (Other)-google<br />conversion_value:    1.000000<br />conversion_type: first_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    2.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.000000<br />conversion_type: first_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:   12.000000<br />conversion_type: first_touch_conversions","channel_name: Social-youtube<br />conversion_value:   14.000000<br />conversion_type: first_touch_conversions","channel_name: Display-google<br />conversion_value:   21.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   42.000000<br />conversion_type: first_touch_conversions","channel_name: Social-other_source<br />conversion_value:   67.000000<br />conversion_type: first_touch_conversions","channel_name: Display-other_source<br />conversion_value:  138.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  131.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  218.000000<br />conversion_type: first_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  216.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1370.000000<br />conversion_type: first_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2132.000000<br />conversion_type: first_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 2662.000000<br />conversion_type: first_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 4429.000000<br />conversion_type: first_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(44,62,80,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"first_touch_conversions","legendgroup":"first_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,2,1,5,9,12,20,44,96,144,225,248,305,1283,2050,1893,5123],"y":[0.8875,1.8875,2.8875,3.8875,4.8875,5.8875,6.8875,7.8875,8.8875,9.8875,10.8875,11.8875,12.8875,13.8875,14.8875,15.8875,16.8875,17.8875,18.8875],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.000000<br />conversion_type: last_touch_conversions","channel_name: (Other)-google<br />conversion_value:    2.000000<br />conversion_type: last_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    1.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.000000<br />conversion_type: last_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:    9.000000<br />conversion_type: last_touch_conversions","channel_name: Social-youtube<br />conversion_value:   12.000000<br />conversion_type: last_touch_conversions","channel_name: Display-google<br />conversion_value:   20.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   44.000000<br />conversion_type: last_touch_conversions","channel_name: Social-other_source<br />conversion_value:   96.000000<br />conversion_type: last_touch_conversions","channel_name: Display-other_source<br />conversion_value:  144.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  225.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  248.000000<br />conversion_type: last_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  305.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1283.000000<br />conversion_type: last_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2050.000000<br />conversion_type: last_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 1893.000000<br />conversion_type: last_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 5123.000000<br />conversion_type: last_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(227,26,28,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"last_touch_conversions","legendgroup":"last_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0.5,1.51515151515152,1.83333333333333,5.01351351351351,10.4353603603604,13.4861281736282,19.2801587301587,45.0039682539683,83.5416274470198,148.451147977188,180.76628462624,237.295185090705,258.901602850109,1331.86985217911,2088.38329821075,2186.5327851989,4847.19060253986],"y":[1.1125,2.1125,3.1125,4.1125,5.1125,6.1125,7.1125,8.1125,9.1125,10.1125,11.1125,12.1125,13.1125,14.1125,15.1125,16.1125,17.1125,18.1125,19.1125],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: linear_touch_conversions","channel_name: (Other)-other_source<br />conversion_value:    0.500000<br />conversion_type: linear_touch_conversions","channel_name: (Other)-google<br />conversion_value:    1.515152<br />conversion_type: linear_touch_conversions","channel_name: Display-(direct)<br />conversion_value:    1.833333<br />conversion_type: linear_touch_conversions","channel_name: Referral-google<br />conversion_value:    5.013514<br />conversion_type: linear_touch_conversions","channel_name: Affiliates-other_source<br />conversion_value:   10.435360<br />conversion_type: linear_touch_conversions","channel_name: Social-youtube<br />conversion_value:   13.486128<br />conversion_type: linear_touch_conversions","channel_name: Display-google<br />conversion_value:   19.280159<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-other_source<br />conversion_value:   45.003968<br />conversion_type: linear_touch_conversions","channel_name: Social-other_source<br />conversion_value:   83.541627<br />conversion_type: linear_touch_conversions","channel_name: Display-other_source<br />conversion_value:  148.451148<br />conversion_type: linear_touch_conversions","channel_name: Referral-other_source<br />conversion_value:  180.766285<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-(direct)<br />conversion_value:  237.295185<br />conversion_type: linear_touch_conversions","channel_name: Paid Search-google<br />conversion_value:  258.901603<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-(direct)<br />conversion_value: 1331.869852<br />conversion_type: linear_touch_conversions","channel_name: Organic Search-google<br />conversion_value: 2088.383298<br />conversion_type: linear_touch_conversions","channel_name: Direct-(direct)<br />conversion_value: 2186.532785<br />conversion_type: linear_touch_conversions","channel_name: Referral-(direct)<br />conversion_value: 4847.190603<br />conversion_type: linear_touch_conversions"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(24,188,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"linear_touch_conversions","legendgroup":"linear_touch_conversions","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,2.60691537761601,4.34485896269336,2.60691537761601,4.34485896269336,8.68971792538671,14.7725204731574,26.0691537761601,44.3175614194722,93.8489535941765,256.346678798908,289.367606915378,345.850773430391,380.609645131938,1531.99727024568,2242.81619654231,2076.84258416742,4134.567788899],"y":[1.3375,2.3375,3.3375,4.3375,5.3375,6.3375,7.3375,8.3375,9.3375,10.3375,11.3375,12.3375,13.3375,14.3375,15.3375,16.3375,17.3375,18.3375,19.3375],"text":["channel_name: (Other)-(direct)<br />conversion_value:    0.000000<br />conversion_type: markov","channel_name: Paid Search-other_source<br />conversion_value:    0.000000<br />conversion_type: markov","channel_name: (Other)-other_source<br />conversion_value:    2.606915<br />conversion_type: markov","channel_name: (Other)-google<br />conversion_value:    4.344859<br />conversion_type: markov","channel_name: Display-(direct)<br />conversion_value:    2.606915<br />conversion_type: markov","channel_name: Referral-google<br />conversion_value:    4.344859<br />conversion_type: markov","channel_name: Affiliates-other_source<br />conversion_value:    8.689718<br />conversion_type: markov","channel_name: Social-youtube<br />conversion_value:   14.772520<br />conversion_type: markov","channel_name: Display-google<br />conversion_value:   26.069154<br />conversion_type: markov","channel_name: Organic Search-other_source<br />conversion_value:   44.317561<br />conversion_type: markov","channel_name: Social-other_source<br />conversion_value:   93.848954<br />conversion_type: markov","channel_name: Display-other_source<br />conversion_value:  256.346679<br />conversion_type: markov","channel_name: Referral-other_source<br />conversion_value:  289.367607<br />conversion_type: markov","channel_name: Paid Search-(direct)<br />conversion_value:  345.850773<br />conversion_type: markov","channel_name: Paid Search-google<br />conversion_value:  380.609645<br />conversion_type: markov","channel_name: Organic Search-(direct)<br />conversion_value: 1531.997270<br />conversion_type: markov","channel_name: Organic Search-google<br />conversion_value: 2242.816197<br />conversion_type: markov","channel_name: Direct-(direct)<br />conversion_value: 2076.842584<br />conversion_type: markov","channel_name: Referral-(direct)<br />conversion_value: 4134.567789<br />conversion_type: markov"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(204,190,147,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"markov","legendgroup":"markov","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":183.37899543379},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Attribution Model","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-256.15,5379.15],"tickmode":"array","ticktext":["0","1000","2000","3000","4000","5000"],"tickvals":[0,1000,2000,3000,4000,5000],"categoryorder":"array","categoryarray":["0","1000","2000","3000","4000","5000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"y","title":{"text":"conversion_value","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other)-(direct)","Paid Search-other_source","(Other)-other_source","(Other)-google","Display-(direct)","Referral-google","Affiliates-other_source","Social-youtube","Display-google","Organic Search-other_source","Social-other_source","Display-other_source","Referral-other_source","Paid Search-(direct)","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other)-(direct)","Paid Search-other_source","(Other)-other_source","(Other)-google","Display-(direct)","Referral-google","Affiliates-other_source","Social-youtube","Display-google","Organic Search-other_source","Social-other_source","Display-other_source","Referral-other_source","Paid Search-(direct)","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"x","title":{"text":"channel_name","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0.33208800332088,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"conversion_type","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"1088475c456f":{"x":{},"y":{},"fill":{},"type":"bar"}},"cur_data":"1088475c456f","visdat":{"1088475c456f":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
# Value
markov_model %>%
    select(-total_conversion) %>%
    rename(markov = total_conversion_value) %>%
    left_join(
        channel_path_heuristic_model %>%
            select_at(vars(channel_name, matches("value")))
    ) %>%
    plot_attribution()
```

```
## Joining, by = "channel_name"
```

<!--html_preserve--><div id="htmlwidget-b08a971005ca120c3252" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-b08a971005ca120c3252">{"x":{"data":[{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,50.61,11.99,455.22,442.53,753.61,1955.17,2258.37,5624.51,17192.57,18199.92,27580.11,25047.94,125247.940000001,214005.6,476648.579999998,520184.050000001],"y":[0.662500000000001,1.6625,2.6625,3.6625,4.6625,5.6625,6.6625,7.6625,8.6625,9.6625,10.6625,11.6625,12.6625,13.6625,14.6625,15.6625,16.6625,17.6625,18.6625],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: (Other)-other_source<br />conversion_value:      0.00000<br />conversion_type: first_touch_value","channel_name: Display-(direct)<br />conversion_value:     50.61000<br />conversion_type: first_touch_value","channel_name: (Other)-google<br />conversion_value:     11.99000<br />conversion_type: first_touch_value","channel_name: Referral-google<br />conversion_value:    455.22000<br />conversion_type: first_touch_value","channel_name: Social-youtube<br />conversion_value:    442.53000<br />conversion_type: first_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    753.61000<br />conversion_type: first_touch_value","channel_name: Display-google<br />conversion_value:   1955.17000<br />conversion_type: first_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2258.37000<br />conversion_type: first_touch_value","channel_name: Social-other_source<br />conversion_value:   5624.51000<br />conversion_type: first_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  17192.57000<br />conversion_type: first_touch_value","channel_name: Display-other_source<br />conversion_value:  18199.92000<br />conversion_type: first_touch_value","channel_name: Referral-other_source<br />conversion_value:  27580.11000<br />conversion_type: first_touch_value","channel_name: Paid Search-google<br />conversion_value:  25047.94000<br />conversion_type: first_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 125247.94000<br />conversion_type: first_touch_value","channel_name: Organic Search-google<br />conversion_value: 214005.60000<br />conversion_type: first_touch_value","channel_name: Direct-(direct)<br />conversion_value: 476648.58000<br />conversion_type: first_touch_value","channel_name: Referral-(direct)<br />conversion_value: 520184.05000<br />conversion_type: first_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(44,62,80,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"first_touch_value","legendgroup":"first_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,0,36.98,133.56,380.24,503.44,719.13,2426.64,2443.44,8456.99,21331.18,23233.75,32291.44,51407.14,115490.880000001,210736.94,335691.439999999,630375.529999995],"y":[0.8875,1.8875,2.8875,3.8875,4.8875,5.8875,6.8875,7.8875,8.8875,9.8875,10.8875,11.8875,12.8875,13.8875,14.8875,15.8875,16.8875,17.8875,18.8875],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: (Other)-other_source<br />conversion_value:      0.00000<br />conversion_type: last_touch_value","channel_name: Display-(direct)<br />conversion_value:     36.98000<br />conversion_type: last_touch_value","channel_name: (Other)-google<br />conversion_value:    133.56000<br />conversion_type: last_touch_value","channel_name: Referral-google<br />conversion_value:    380.24000<br />conversion_type: last_touch_value","channel_name: Social-youtube<br />conversion_value:    503.44000<br />conversion_type: last_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    719.13000<br />conversion_type: last_touch_value","channel_name: Display-google<br />conversion_value:   2426.64000<br />conversion_type: last_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2443.44000<br />conversion_type: last_touch_value","channel_name: Social-other_source<br />conversion_value:   8456.99000<br />conversion_type: last_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  21331.18000<br />conversion_type: last_touch_value","channel_name: Display-other_source<br />conversion_value:  23233.75000<br />conversion_type: last_touch_value","channel_name: Referral-other_source<br />conversion_value:  32291.44000<br />conversion_type: last_touch_value","channel_name: Paid Search-google<br />conversion_value:  51407.14000<br />conversion_type: last_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 115490.88000<br />conversion_type: last_touch_value","channel_name: Organic Search-google<br />conversion_value: 210736.94000<br />conversion_type: last_touch_value","channel_name: Direct-(direct)<br />conversion_value: 335691.44000<br />conversion_type: last_touch_value","channel_name: Referral-(direct)<br />conversion_value: 630375.53000<br />conversion_type: last_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(227,26,28,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"last_touch_value","legendgroup":"last_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,24.43125,57.225,58.1369696969697,418.016351351351,487.855980241605,739.025952702703,2078.98149206349,2614.68153174603,7544.37124714773,22960.7187020122,21841.8934948129,33039.408699502,31427.8168779343,121494.924251787,210524.328621619,392847.101720134,587499.801857245],"y":[1.1125,2.1125,3.1125,4.1125,5.1125,6.1125,7.1125,8.1125,9.1125,10.1125,11.1125,12.1125,13.1125,14.1125,15.1125,16.1125,17.1125,18.1125,19.1125],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: linear_touch_value","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: linear_touch_value","channel_name: (Other)-other_source<br />conversion_value:     24.43125<br />conversion_type: linear_touch_value","channel_name: Display-(direct)<br />conversion_value:     57.22500<br />conversion_type: linear_touch_value","channel_name: (Other)-google<br />conversion_value:     58.13697<br />conversion_type: linear_touch_value","channel_name: Referral-google<br />conversion_value:    418.01635<br />conversion_type: linear_touch_value","channel_name: Social-youtube<br />conversion_value:    487.85598<br />conversion_type: linear_touch_value","channel_name: Affiliates-other_source<br />conversion_value:    739.02595<br />conversion_type: linear_touch_value","channel_name: Display-google<br />conversion_value:   2078.98149<br />conversion_type: linear_touch_value","channel_name: Organic Search-other_source<br />conversion_value:   2614.68153<br />conversion_type: linear_touch_value","channel_name: Social-other_source<br />conversion_value:   7544.37125<br />conversion_type: linear_touch_value","channel_name: Paid Search-(direct)<br />conversion_value:  22960.71870<br />conversion_type: linear_touch_value","channel_name: Display-other_source<br />conversion_value:  21841.89349<br />conversion_type: linear_touch_value","channel_name: Referral-other_source<br />conversion_value:  33039.40870<br />conversion_type: linear_touch_value","channel_name: Paid Search-google<br />conversion_value:  31427.81688<br />conversion_type: linear_touch_value","channel_name: Organic Search-(direct)<br />conversion_value: 121494.92425<br />conversion_type: linear_touch_value","channel_name: Organic Search-google<br />conversion_value: 210524.32862<br />conversion_type: linear_touch_value","channel_name: Direct-(direct)<br />conversion_value: 392847.10172<br />conversion_type: linear_touch_value","channel_name: Referral-(direct)<br />conversion_value: 587499.80186<br />conversion_type: linear_touch_value"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(24,188,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"linear_touch_value","legendgroup":"linear_touch_value","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"h","width":[0.225,0.225,0.225,0.225,0.225,0.225,0.225,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001,0.225000000000001],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,0,506.187059088406,75.432643154097,353.985727065381,737.480637745281,492.378487228294,719.215814028871,3851.61786762212,3575.27078051741,6411.01536261435,43071.609264254,34803.0569266544,37912.617080583,57789.2385626558,162484.728529391,241035.955840527,352606.683146674,489232.246270149],"y":[1.3375,2.3375,3.3375,4.3375,5.3375,6.3375,7.3375,8.3375,9.3375,10.3375,11.3375,12.3375,13.3375,14.3375,15.3375,16.3375,17.3375,18.3375,19.3375],"text":["channel_name: (Other)-(direct)<br />conversion_value:      0.00000<br />conversion_type: markov","channel_name: Paid Search-other_source<br />conversion_value:      0.00000<br />conversion_type: markov","channel_name: (Other)-other_source<br />conversion_value:    506.18706<br />conversion_type: markov","channel_name: Display-(direct)<br />conversion_value:     75.43264<br />conversion_type: markov","channel_name: (Other)-google<br />conversion_value:    353.98573<br />conversion_type: markov","channel_name: Referral-google<br />conversion_value:    737.48064<br />conversion_type: markov","channel_name: Social-youtube<br />conversion_value:    492.37849<br />conversion_type: markov","channel_name: Affiliates-other_source<br />conversion_value:    719.21581<br />conversion_type: markov","channel_name: Display-google<br />conversion_value:   3851.61787<br />conversion_type: markov","channel_name: Organic Search-other_source<br />conversion_value:   3575.27078<br />conversion_type: markov","channel_name: Social-other_source<br />conversion_value:   6411.01536<br />conversion_type: markov","channel_name: Paid Search-(direct)<br />conversion_value:  43071.60926<br />conversion_type: markov","channel_name: Display-other_source<br />conversion_value:  34803.05693<br />conversion_type: markov","channel_name: Referral-other_source<br />conversion_value:  37912.61708<br />conversion_type: markov","channel_name: Paid Search-google<br />conversion_value:  57789.23856<br />conversion_type: markov","channel_name: Organic Search-(direct)<br />conversion_value: 162484.72853<br />conversion_type: markov","channel_name: Organic Search-google<br />conversion_value: 241035.95584<br />conversion_type: markov","channel_name: Direct-(direct)<br />conversion_value: 352606.68315<br />conversion_type: markov","channel_name: Referral-(direct)<br />conversion_value: 489232.24627<br />conversion_type: markov"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(204,190,147,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"markov","legendgroup":"markov","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":183.37899543379},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Attribution Model","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-31518.7764999998,661894.306499995],"tickmode":"array","ticktext":["0e+00","2e+05","4e+05","6e+05"],"tickvals":[0,200000,400000,600000],"categoryorder":"array","categoryarray":["0e+00","2e+05","4e+05","6e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"y","title":{"text":"conversion_value","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.4,19.6],"tickmode":"array","ticktext":["(Other)-(direct)","Paid Search-other_source","(Other)-other_source","Display-(direct)","(Other)-google","Referral-google","Social-youtube","Affiliates-other_source","Display-google","Organic Search-other_source","Social-other_source","Paid Search-(direct)","Display-other_source","Referral-other_source","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"categoryorder":"array","categoryarray":["(Other)-(direct)","Paid Search-other_source","(Other)-other_source","Display-(direct)","(Other)-google","Referral-google","Social-youtube","Affiliates-other_source","Display-google","Organic Search-other_source","Social-other_source","Paid Search-(direct)","Display-other_source","Referral-other_source","Paid Search-google","Organic Search-(direct)","Organic Search-google","Direct-(direct)","Referral-(direct)"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"x","title":{"text":"channel_name","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0.33208800332088,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"conversion_type","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"1088582a7a44":{"x":{},"y":{},"fill":{},"type":"bar"}},"cur_data":"1088582a7a44","visdat":{"1088582a7a44":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

----------


## 7. Conclusion

It was a great introdction to Google Merchandise of Marketing Attribution with use of heuristics and Markov Chain. 

------------


## 8. Next Steps

1. We will the Split Paths by Transactions
2. Data Visualization of paths
3. Use of Machine Learning - <http://www0.cs.ucl.ac.uk/staff/w.zhang/rtb-papers/data-conv-att.pdf>
