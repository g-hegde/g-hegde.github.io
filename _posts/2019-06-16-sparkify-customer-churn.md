---
layout: post
title: A Predictive Model of Customer Churn in the Spark Big-Data Framework
---

![Sparkify](/images/Sparkify.png)

<i>Sparkify - a fictional music app. Image courtesy of Udacity.</a></i>

# Table of Contents  
1. [Introduction](#intro)  
2. [Problem Statement](#prob-stat)  
3. [Solution Strategy and expected result](#strat)  
4. [Performance metrics](#metrics) 
5. [Loading libraries and data](#load)
5. [Churn Definition](#churn)  
6. [Data Exploration and Visualization](#eda)  
7. [Feature Engineering](#feng)
8. [Model implementation and optimization](#model)  
9. [Conclusions](#conclusions)  

<a name="intro"></a>  
## Introduction  

Churn is a problem that companies face on a regular basis. Customers may drop a service for a variety of reasons. They may find competitors better. They may find cost prohibitive. They may simply forget to renew a subscription. Whatever the reason may be, customer churn is an undesirable aspect of doing business and companies like to avoid it.  

In this project, a fictional music app called Sparkify is set up with user data. The aim in this project is to use customer data to create a predictive model of customer churn in the Spark big-data framework. If companies are able to utilize customer-usage data and find patterns in them and map them accurately to indicate which customers might churn, they could, in principle, incentivize customers to stay with them. Finding such patterns could also be useful from the perspective of targeted advertising - customers demographics with advantageous average behavior can be targeted for advertising with hopes of better returns than blanket advertising.  

The write-up that follows is accompanied by <a href = "https://github.com/g-hegde/spark-customer-churn/blob/master/Sparkify.ipynb">a Jupyter notebook</a> with detailed documentation. 

<a name="prob-stat"></a>
## Problem Statement  

Using Spark and associated libraries and customer data for Sparkify, a fictitious music app, create and validate a predictive model for customer churn.  

<a name="strat"></a>  
## Solution Strategy and expected result  

Firstly,  churn is defined in the context of the problem statement. Next, differences in customer demographic (location, gender) and usage (time spent, engagement) between groups of customers that churn versus those that don't churn are explored. The <i>a priori</i> expectation is that some of these features will reveal a substantial difference between customers that churn versus those that don't. This information is used to create useful features for a classification model for churn. A simple model with a minimum number of features that can easily be interpreted is first attempted and is optimized for performance.  

<a name="metrics"></a>  
## Performance Metrics  

Since the aim is predict customers that churn versus those that don't, the problem is one of binary classification. The predictive classification mmodel developed here is evaluated using standard metrics for binary output data - accuracy and F1-score. F1-score is given greater importance from an interpretation perspective due to imbalanced nature of the output data (significantly fewer customers churn than don't).  

<a name="load"></a>  
## Loading libraries and datasets  

Since the analysis is performed in Spark, the primary libraries loaded are pyspark.sql (for data munging) and pyspark.ml (for machine learning). I also found that it was more efficient to perform data exploration in Pandas since I used a smaller version (~150 MB) of Sparkify data that easily fits in the memory of a single standard machine. Additional libraries for plotting (matplotlib, seaborn as needed), datetime, time are also needed.  

<a name="churn"></a>  
## Churn Definition  

Churn is defined at a user level in this dataset as the subset of users for which there is a single cancellation event. For such users churn=1. For all other users, churn=0. An initial bit of data-munging in Spark SQL is required to create the churn column corresponding to all users that cancelled service.

<a name="eda"></a>  
## Data Exploration and Visualization  
A birds-eye-view exploration of the data set reveals that there are 226 users in total. The datasets are made of the following columns  
![Schema](/images/schema.PNG)  
<i>Columns in the Sparkify customer dataset. Dataset courtesy of Udacity.</a></i>

The column relevant to determining whether a user churns or not is the 'page' column. The column essentially describes all actions a user takes on the platform. From the point a user registers, to the point that the user cancels service, each event is logged in the page column. A brief description of the page column and it's associated values is given in the figure below.  

![Page](/images/page.png)  
<i>'page' column in the Sparkify dataset and all the unique values it can take.</a></i>

Exploring the 'page' and 'userId' columns a little further, I found that a total of 52 members cancelled service or churned (per our definition above). The churn/non-churn ratio is therefore quite low. Some elementary data-munging in Pandas (and/or Spark) allows us to convert this information into a separate column named 'churn' in the dataset. Essentially, it involves identifying every user associated with a cancellation event and marking all rows corresponding to this user in the 'churn' column to be equal to 1. For all other users the 'churn' column for all rows is equal to 0.  

Having obtained this information one can start exploring which columns (or columns derived from these columns) show marked differences in values for users that churn versus those that don't. Columns that show large differences in churn behavior are good candidates for inclusion as features in the machine learning task that follows.  

Columns related to 'firstname' or 'artist' should have no predictive power and are not explored further. 'auth' has cancelation information, but not as detailed as 'page'. 'method' (probably) has information related to system actions - 'PUT' when information is added to the database, and 'GET' when information is accessed. This field showed insignificant difference in the distribution for churned vesus non-churned users. Similarly 
The first column that seems <i>a priori</i> should show some difference between the two populations is 'gender'. The figure below indicates that almost 8% more men churned as compared to women. Without hypothesis testing it is infeasible to conclude if this difference is statistically significant. But since the different <i>seems</i> to be important, we could consider it's inclusion as a feature of interest.

![Gender](/images/gender_churn.png)

The next promising column is 'itemInSession'. The figure below indicates that this field shows insignificant difference between the two populations.

![itemInSession](/images/itemInSession_churn.png)

Similarly, 'length' shows insignificant difference between the two populations.

![length](/images/length_churn.png)





<a name="feng"></a>
## Feature Engineering  

<a name="model"></a>  
## Model Implementation and optimization  

<a name="conclusions"></a>  
## Conclusions  
