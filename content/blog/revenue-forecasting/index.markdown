---
title: "Bootstrapped Resampling Regression & Revenue Forecasting"
subtitle: ""
excerpt: "We'll analyze regression coefficients with bootstrapped resamples, visualize most important features and forecast future revenue"
date: 2022-11-28
author: "Alex Labuda"
draft: false
thumbnail_left: true # for list-sidebar only
show_author_byline: true
images: 
series:
tags: ["machine learning", "regression", "feature importance", "time-series forecasting"]
categories:
layout: single
---



![](featured.jpg)

In this post, I'll begin by visualizing marketing related time-series data which includes revenue and spend for several direct marketing channels.

I will fit a simple regression model and examine the effect of each channel on revenue.

In order to better understand the reliability of each coefficient, I will fit many bootstrapped resamples to develop a more robust estimate of each coefficient.

Lastly, I will create a forecasting model to predict future revenue. Enjoy!



# The Data





|date       |   revenue|  tv_spend| billboard_spend| print_spend| facebook_impressions|
|:----------|---------:|---------:|---------------:|-----------:|--------------------:|
|2018-11-23 | 2,754,372| 167,687.6|               0|   95,463.67|           72,903,853|
|2018-11-30 | 2,584,277| 214,600.9|               0|        0.00|           16,581,100|
|2018-12-07 | 2,547,387|       0.0|         248,022|    3,404.00|           49,954,774|
|2018-12-14 | 2,875,220| 625,877.3|               0|  132,600.00|           31,649,297|
|2018-12-21 | 2,215,953|       0.0|         520,005|        0.00|            8,802,269|
|2018-12-28 | 2,569,922| 249,189.8|               0|  239,417.33|           49,902,081|


# Visualize the Data

First lets visualize our target variable, `revenue`

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="768" />

# Fit: Simple Regression Model

First we'll start by fitting a simple regression model to the data. This is just the grand mean of our data.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="768" />

We can make a more accurate model by using month of year as independent variables for our model

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="768" />

We can also introduce our spend variables as input to our model to study each channel's effect on revenue

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="768" />

# A Simple Linear Model

Now lets take a look at our simple model output that includes direct marketing features and examine their effect of revenue

We can see that:

-   TV spend, print spend, competitor sales and Facebook spend are statistically significant
-   Print spend has the largest positive effect at increasing revenue of each of the direct marketing inputs
-   A large positive intercept suggests that this company already has a strong baseline of sales


|term             |   estimate|   std.error| statistic| p.value|
|:----------------|----------:|-----------:|---------:|-------:|
|(Intercept)      | 734516.829| 1069857.846|     0.687|   0.493|
|as.numeric(date) |    -34.162|      57.542|    -0.594|   0.553|
|tv_spend         |      0.501|       0.091|     5.521|   0.000|
|billboard_spend  |      0.042|       0.122|     0.344|   0.731|
|print_spend      |      0.876|       0.395|     2.216|   0.028|
|search_spend     |      0.536|       0.668|     0.803|   0.423|
|facebook_spend   |      0.358|       0.210|     1.707|   0.089|
|competitor_sales |      0.287|       0.012|    24.846|   0.000|

## Viz: SLM coefficients

We can plot the coefficient and confidence intervals to make it easier to see the reliability of each estimate

-   Print spend has the largest positive coefficient, but also the widest confidence interval
-   Our model is very confident in the estimated effect of competitor sales

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="768" />

# Fit: Bootstrap Resampling

## How reliable are our coefficients?

We can fit a model using bootstrapped resamples of our data. This allows us to determine the stability of our coefficient estimates. Essentially this is fitting many small models to examine the variation of each channel's coefficient.

-   By default `reg_intervals` uses 1,001 bootstrap samples for t-intervals and 2,001 for percentile intervals.


|term             |    .lower| .estimate|  .upper|
|:----------------|---------:|---------:|-------:|
|as.numeric(date) | -132.5731|  -34.5793| 67.0201|
|billboard_spend  |   -0.1206|    0.0397|  0.1667|
|competitor_sales |    0.2641|    0.2864|  0.3042|
|facebook_spend   |   -0.0361|    0.3629|  0.6992|
|print_spend      |    0.2611|    0.8716|  1.4596|
|search_spend     |   -0.5765|    0.5603|  1.7128|
|tv_spend         |    0.2153|    0.5088|  0.7818|

## Viz: Bootstrapped Resampled Coefficients

### Crossbar chart

-   Here we can see the wide confidence interval surrounding `search_spend`.
-   This makes sense since the coefficient for this channel in our first linear model was not statistically significant.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="768" />

# Time-Series Forecasting

Lets build our forecasting model!

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="768" />

## Training & Testing Splits

First, I'll split the data into training and testing sets

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="768" />

## Modeling

Now we can create the recipe for our model. 

After we `bake`, we can see what the output looks like.


```
## Rows: 6
## Columns: 29
## $ date           <date> 2018-11-23, 2018-11-30, 2018-12-07, 2018-12-14, 2018-1…
## $ revenue        <dbl> 2754372, 2584277, 2547387, 2875220, 2215953, 2569922
## $ date_index.num <dbl> 1542931200, 1543536000, 1544140800, 1544745600, 1545350…
## $ date_year      <int> 2018, 2018, 2018, 2018, 2018, 2018
## $ date_year.iso  <int> 2018, 2018, 2018, 2018, 2018, 2018
## $ date_half      <int> 2, 2, 2, 2, 2, 2
## $ date_quarter   <int> 4, 4, 4, 4, 4, 4
## $ date_month     <int> 11, 11, 12, 12, 12, 12
## $ date_month.xts <int> 10, 10, 11, 11, 11, 11
## $ date_month.lbl <ord> November, November, December, December, December, Decem…
## $ date_day       <int> 23, 30, 7, 14, 21, 28
## $ date_hour      <int> 0, 0, 0, 0, 0, 0
## $ date_minute    <int> 0, 0, 0, 0, 0, 0
## $ date_second    <int> 0, 0, 0, 0, 0, 0
## $ date_hour12    <int> 0, 0, 0, 0, 0, 0
## $ date_am.pm     <int> 1, 1, 1, 1, 1, 1
## $ date_wday      <int> 6, 6, 6, 6, 6, 6
## $ date_wday.xts  <int> 5, 5, 5, 5, 5, 5
## $ date_wday.lbl  <ord> Friday, Friday, Friday, Friday, Friday, Friday
## $ date_mday      <int> 23, 30, 7, 14, 21, 28
## $ date_qday      <int> 54, 61, 68, 75, 82, 89
## $ date_yday      <int> 327, 334, 341, 348, 355, 362
## $ date_mweek     <int> 4, 5, 2, 3, 4, 5
## $ date_week      <int> 47, 48, 49, 50, 51, 52
## $ date_week.iso  <int> 47, 48, 49, 50, 51, 52
## $ date_week2     <int> 1, 0, 1, 0, 1, 0
## $ date_week3     <int> 2, 0, 1, 2, 0, 1
## $ date_week4     <int> 3, 0, 1, 2, 3, 0
## $ date_mday7     <int> 4, 5, 2, 3, 4, 5
```

### Preprocessing Steps

Now we'll add a few additional preprocessing steps to our recipe:

-   Convert a date column into a fourier series
-   Remove date
-   Normalize the inputs
-   one-hot encode categorical inputs (add dummies)


```
## Rows: 6
## Columns: 39
## $ revenue           <dbl> 2754372, 2584277, 2547387, 2875220, 2215953, 2569922
## $ date_index.num    <dbl> -1.712915, -1.690799, -1.668683, -1.646568, -1.62445…
## $ date_year         <dbl> -2.142581, -2.142581, -2.142581, -2.142581, -2.14258…
## $ date_half         <int> 2, 2, 2, 2, 2, 2
## $ date_quarter      <int> 4, 4, 4, 4, 4, 4
## $ date_month        <int> 11, 11, 12, 12, 12, 12
## $ date_day          <int> 23, 30, 7, 14, 21, 28
## $ date_second       <int> 0, 0, 0, 0, 0, 0
## $ date_wday         <int> 6, 6, 6, 6, 6, 6
## $ date_mday         <int> 23, 30, 7, 14, 21, 28
## $ date_qday         <int> 54, 61, 68, 75, 82, 89
## $ date_yday         <int> 327, 334, 341, 348, 355, 362
## $ date_mweek        <int> 4, 5, 2, 3, 4, 5
## $ date_week         <int> 47, 48, 49, 50, 51, 52
## $ date_week2        <int> 1, 0, 1, 0, 1, 0
## $ date_week3        <int> 2, 0, 1, 2, 0, 1
## $ date_week4        <int> 3, 0, 1, 2, 3, 0
## $ date_mday7        <int> 4, 5, 2, 3, 4, 5
## $ date_sin365_K1    <dbl> -0.06634888, -0.04916362, -0.03196379, -0.01475450, …
## $ date_cos365_K1    <dbl> 0.9977965, 0.9987907, 0.9994890, 0.9998911, 0.999997…
## $ date_month.lbl_01 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_02 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_03 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_04 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_05 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_06 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_07 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_08 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_09 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_10 <dbl> 0, 0, 0, 0, 0, 0
## $ date_month.lbl_11 <dbl> 1, 1, 0, 0, 0, 0
## $ date_month.lbl_12 <dbl> 0, 0, 1, 1, 1, 1
## $ date_wday.lbl_1   <dbl> 0, 0, 0, 0, 0, 0
## $ date_wday.lbl_2   <dbl> 0, 0, 0, 0, 0, 0
## $ date_wday.lbl_3   <dbl> 0, 0, 0, 0, 0, 0
## $ date_wday.lbl_4   <dbl> 0, 0, 0, 0, 0, 0
## $ date_wday.lbl_5   <dbl> 0, 0, 0, 0, 0, 0
## $ date_wday.lbl_6   <dbl> 1, 1, 1, 1, 1, 1
## $ date_wday.lbl_7   <dbl> 0, 0, 0, 0, 0, 0
```

## Model Specs

1. linear regression
2. xgboost
3. prophet


```
## Linear Regression Model Specification (regression)
## 
## Computational engine: lm
```

```
## Boosted Tree Model Specification (regression)
## 
## Computational engine: xgboost
```

```
## PROPHET Regression Model Specification (regression)
## 
## Computational engine: prophet
```

## Workflow

We can add our recipe and model to a workflow


```
## ══ Workflow ════════════════════════════════════════════════════════════════════
## Preprocessor: Recipe
## Model: prophet_reg()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────
## 1 Recipe Step
## 
## • step_timeseries_signature()
## 
## ── Model ───────────────────────────────────────────────────────────────────────
## PROPHET Regression Model Specification (regression)
## 
## Computational engine: prophet
```

## Fit our Model


```
## ══ Workflow [trained] ══════════════════════════════════════════════════════════
## Preprocessor: Recipe
## Model: prophet_reg()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────
## 1 Recipe Step
## 
## • step_timeseries_signature()
## 
## ── Model ───────────────────────────────────────────────────────────────────────
## PROPHET w/ Regressors Model
## - growth: 'linear'
## - n.changepoints: 25
## - changepoint.range: 0.8
## - yearly.seasonality: 'auto'
## - weekly.seasonality: 'auto'
## - daily.seasonality: 'auto'
## - seasonality.mode: 'additive'
## - changepoint.prior.scale: 0.05
## - seasonality.prior.scale: 10
## - holidays.prior.scale: 10
## - logistic_cap: NULL
## - logistic_floor: NULL
## - extra_regressors: 32
```



## Measure Accuracy


| .model_id|.model_desc           |     rmse|   rsq|
|---------:|:---------------------|--------:|-----:|
|         1|LM                    | 313485.5| 0.804|
|         2|XGBOOST               | 352541.8| 0.765|
|         3|PROPHET W/ REGRESSORS | 386075.2| 0.788|


## Forcasting Results

Lets visualize each model's prediction accuracy

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-21-1.png" width="768" />




**Sources:** *Matt Dancho, Julia Silge*
