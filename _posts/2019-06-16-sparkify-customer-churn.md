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


'level' contains information regarding the type of account. This is potentially a promising 'column'. Once can conceive that the usage patterns of users with free accounts may be different than users with paid accounts. Indeed, the figure below shows <i>some</i> difference between the two populations. Similar to 'gender' ~8% more of free account users churn as compared to paid account users. This is somewhat counterintuitive (and potentially worrisome from a product perspective), since one would not expect free-account users to cancel.  

![level](/images/level_churn.png)  

'location' is one of the most interesting columns in the dataset. The figure below shows location versus churn information for the first few locations, arranged alphabetically. Certain locations show 100% churn, while some show 0%. This behavior is counter-intuitive as one would not expect 100% churn in any locations <i>a priori </i>. One does not expect usage patterns to be so consistent across samples of populations in particular geographic areas since one expects churn to be highly dependent on individual customers and their actions.  

![location](/images/location_churn.png)  

'location' thus seems to be a feature interesting enough to be included in the feature set for modeling. There are a number of ways to approach the inclusion of 'location'. A standard way would be to one-hot encode all 114 locations, but this would unnecessarily increase dimensionality of the dataset without necessarily providing greater insight. Another approach is to bin location-dependent churn into buckets where churn is low, medium and high. This reduces 114 potential variables to 3. The problem with the former approach is that it does not generalize beyond the locations provided and the problem with the second-approach (and to some extent, the first) is that both directly include information about churn. We can flag this for potential data-leakage and continue with our analysis. If inclusion of this variable leads to unexpectedly high performance, we can take this to be an indication of data-leakage.  

Among other columns, only 'registration', 'ts' and 'page' columns contain information that is relevant to modeling since they reveal usage patterns. These columns cannot be used directly, however and their inclusion in the feature set is discussed in the next section on feature engineering.  

<a name="feng"></a>
## Feature Engineering  
All feature engineering is performed with PySpark. From the columns selected for inclusion in the feature set, 'gender' and 'level' are fairly straightforward to encode into features on the basis of one-hot encoding (indexing in Spark). To convert 'registration' and 'ts' columns into intuitive features, these columns are first converted from Unix Timestamp to Datetime objects. Subsequently, the dataset is grouped by userId and the timestamp ('ts') corresponding to the most recent user action is converted to a Datetime object. The maximum time spent by a user on the platform is then computed as the difference in time between the last action and the registration. This difference is stored as 'timeSpentMax' column and it's distribution for churned versus non-churned users is shown below.  

![timespentmax](/images/timespent_churn.png)  

A fairly large difference is seen in the distributions of 'timeSpentMax' for the two populations. The median value for churned users is lower than that for non-churned users. the expectation is that on -average, the total time spent on the platform should have something useful to say about churn behavior.  
 
Next, a 'user_engagement' column is engineered by grouping the dataset by 'userId' and simply counting the total number of actions taken by a user on the platform in the 'page' column. The expectation here is that more engaged users should churn less. Visualizing this feature versus churn, one can see that there is again a significant difference in engagement behavior between churned and non-churned users. Not only are users who churn engaged less than users who don't, the maximum engagement for churned users is also capped at ~4000. This feature is therefore potentially useful.  

![user_engagement](/images/user_engage_churn.png)  

'location' information is binned into 'high', 'medium' and 'low' indicating the proportion of users that churn from the location that the user belongs to. This variable is then indexed in Spark as a column called 'location_churn' for inclusion in the feature dataset.  

Finally, a column called 'time_engage_prod' is engineered by multiplying the user_engagement column with the 'timeSpentmax' column. The intuition here is that while the individual columns may reveal significant differences in churn behavior, the product should further separate out these distributions. Intuitively, users that are both engaged more and have spent a lot of time on the platform should be far less likely to churn than users that spend very little time on the platform and engage less. Similarly, users that engage less and have spent more time on the platform should be more likely to churn than users that engage more and have spent less time on the platform. The distribution of this column does reveal sharper differences in the two populations as shown below.

![time_engage_prod](/images/eng_time_prod_churn.png)  


<a name="model"></a>  
## Model Implementation and optimization  
The columns 'gender', 'level', 'location_churn', 'timeSpentMax', 'user_engagement' and 'time_engage_prod' are included in the machine learning Pipeline as features, while 'churn' is used as the output/label variable. The full dataset is randomly split into 'train' and 'test' subsets for model fitting and evaluation.  

Since the label takes two distinct values - 1 for churn and 0 for non-churn, the problem is cast as a classification problem. Initially 3 classification models are explored  - Logistic Regression (LR), Random Forest-based Classification (RFC) and Gradient-Boosted Classification (GBC). The native implementation of each of these 3 algorithms in Spark is used to evaluate accuracy, F1-score and time taken to fit and evaluate the algorithms.

RFC provides the best tradeoff-between execution time and accuracy. The default RFC provides an accuracy and F1-score of ~97%, which is quite impressive for a model with just 6 features.  

The RFC is further optimized by creating a parameter grid with 3 parameters - 'minInfoGain', 'maxDepth' and 'numTrees'. As of this first attempt, the range-sweep for each of these parameters was limited to two values. Simple 3-fold cross validation (CV) was then performed using this parameter grid. CV provided a marginal improvement (~0.5%) in the F1-score.

The best performing algorithm - GBC - gave an accuracy of nearly 99.2%. Obtaining such high accuracy from a fairly complex dataset recalls the aforementioned suspicion of data-leakage. When 'location_churn' is excluded from the feature set and the same modeling proceedure is repeated, one sees a significant reduction in the performance of LR. RFC and GBC, however still show impressive performances of ~93% and ~99% respectively.

The variance in the data is almost entirely (~97% of 93%) explained by 'timeSpentMax','user_engagement' and 'time_engage_prod' features. In spite of being a fairly simple model with only 5 variables, the model has an F1-score of nearly 93%. In terms of interpretability, the main three features indicate that a user's churn behaviour can almost entirely be described by the net time spent on the platform and the total number of interactions had on the platform. 

Since hyperparameter tuning performed above on the extended feature set (with location_churn as a feature) provided marginal improvement in the performance of the Random Forest Classifier, I am choosing to not repeat the process for the reduced feature set. The default parameters of the Random Forest classifier - max information gain of 0.0, maximum tree depth of 5 and use of 20 trees in the forest are retained as the final parameters of the model.  

<a name="conclusions"></a>  
## Conclusions  

Sparkify user churn behavior was shown to be strongly dependent on the net time spent on the platform and the number of interactions had by the user in that time. A F1-score metric of close to 93% was attained by the Random Forest Classifier with default parameters on test data. Parameter tuning (limited, due to execution time constraints) improved performance, but insignificantly.

The markers of user behavior and their interaction with the product are fairly intuitive in hindsight, but it is helpful that the modeling seems to confirm such an insight. The feature set that leaked churn data into features used for machine learning was discarded in favor of the feature set with just 5 features - gender, payment level, time spent on the platform, user engagement and the product of engagement and time spent.  

While I deliberately selected Pandas for EDA (and provided what I thought was a reasonable justification for it's use in this case), the use of Pandas and Pyspark make the overall 'user' experience for a person browsing the project a little clunky. This could definitely be improved.  

Another area of improvement is hyperparameter training. Model optimization showed marginal improvement and that was probably because only a few parameters were chosen to optimize. In addition, only a small range of the possible sweep in parameters was explored. Time permitting, this is definitely an area of improvement.  

The area I feel needs the most improvement though, is the modeling approach. Using a vanilla classifier in this project seems reasonable since results justify them. What would be really interesting would be attempting models of user action. Based on the history of actions a specific user has taken, what is the probability that a user might take X, Y or Z action next? The methodsused in this write-up don't really apply and a Bayesian /Probabilistic Graphical Model based analysis is needed for such problems. This is beyond the scope of the present project, however.  

At the attained level of performance, it seems un-necessary to involve more complicated learning algorithms. Even a simple Random Forest classifier seems to provide reasonable performance metrics. A more thorough analysis of learning algorithm suitability would involve an exploration of the trade-offs in play. For instance, is it worth it to use a Gradient boosted classifier with 2-3% improved accuracy at the cost of nearly double training time and prediction time? What is the net-revenue add from this improved performance? These are questions of business relevance that can have an impact on model decisions. If a significant improvement in revenue results from the added 3-5% improvement in performance from Gradient boosting and the time for prediction can be cut down significantly (presumably when truly distributed), then gradient boosting is clearly the better choice.  

The insight regarding location and its relevance to predicting churn must also be highlighted - while I eventually discarded this feature, the feature gave some revealing insights - some locations experienced 100% churn and others experienced zero. I would not have expected such a large number of locations to show extremes. Perhaps this has to do with the fact that the dataset is synthetic, but it would be interesting to know if such an extreme distribution exists in reality. Such information could be of potential use for a customer database that is geographically confined - for instance if the list of locations does not expand beyond what is already present, such information can be used fruitfully to incentivize users in high-churn locations. Alternately, a cost-benefit analysis might conclude that particular locations are not worth incentivizing due to low overall revenue.  

On a personal level, my familiarity and comfort with Pandas, coupled with its ease of use and execution speed for small datasets made my learning curve for PySpark fairly steep. The power of Spark would probably show up in larger datasets and spinning-up a Spark cluster to see this power in action is something I hope to do soon.  
