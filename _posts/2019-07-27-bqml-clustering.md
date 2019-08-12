---
layout:     post
title:      "Customer Segmentation with BigQuery ML"
date:       2019-07-27 12:00:00
author:     "Jason Feng"
header-img: "img/railway-track-2049394_960_720.jpg"
excerpt_separator: <!--more-->
tags:
    - GCP
    - BigQuery
    - K-means
---

Customer segmentation is the marketing strategy that divides customers into different groups based on some specific ways of similarity. Marketing teams can tailor their content and media to unique audiences according to the segmentations. BigQuery, the analytics data warehouse on Google Cloud, now enables users to create and execute machine learning models with standard SQL to address such solution easily and quickly.

<!--more-->
### Introduction
BigQuery ML is a capability inside BigQuery that allows data scientists and analysts to build and operationalize machine learning models in minutes on massive structured or semi-structured datasets. BigQuery ML democratizes predictive analytics so that data analysts unfamiliar with programming languages like Python and Java can build machine learning models with basic SQL queries.

### Solution
Just following the steps below, we can create the K-means model to build customer segmentations. All is done within the BigQuery console.

#### Step 1: Create the model

With SQL-like queries, we can create and train the k-means clustering model. We just need to provide is the k (number of clusters), distance metric, and features.
```SQL
CREATE MODEL
  my_dataset.customer_segmentation_1
OPTIONS
  ( model_type = 'kmeans',
    num_clusters = 3,
    distance_type = 'euclidean') AS
SELECT
  Age,
  Income,
  CCAvg,
  Mortgage
FROM
  my_dataset.abcbank
```

#### Step 2: Evaluate the model
The evaluation provides metrics as [Davies–Bouldin index](https://en.wikipedia.org/wiki/Davies%E2%80%93Bouldin_index) and Mean squared distance.
```SQL
SELECT
  *
FROM
  ML.EVALUATE(model my_dataset.customer_segmentation_1)
```
![](/img/bqml-evaluation-190727.png)

#### Step 3: Make prediction with the trained model
We can then get the clustering result for each observation. 
```SQL
SELECT
  *
FROM
  ML.PREDICT(model my_dataset.customer_segmentation_1,
    table my_dataset.abcbank)

SELECT
  centroid_id, ID, Age, Income, CCAvg, Mortgage
FROM
  ML.PREDICT(model my_dataset.customer_segmentation_1,
    table my_dataset.abcbank)
```
The output looks like below.
![](/img/bqml-customer-seg-190727.png)

### K-means under the hood

K-means is an unsupervised machine learning technique which works on datasets without labels. K-means is one of the most popular "clustering" algorithms. It will group data into clusters which are similar to one another.

Here are the steps to implement k-means clustering algorithm.

1. Specify the value of k (number of clusters), and the distance metric (i.e. Euclidean distance).

2. Initialise k centroids.

3. For each observation, calculate the distance to each centroid. Then assign the observation to the cluster which the distance from the observation to the centroid is the shortest.

4. For each cluster, re-calculate the new centroid which is the center (mean) of all the observations in each cluster.

5. Repeat step 3 and 4, until the result is converged or reached to the pre-defined iteration.

The above approach is called Expectation-Maximization. Step 3 is the E-step, and step 4 is the M-step. There are some variants for step 2 to initialise the centroids (i.e. k-means++) which will lead to speed up the whole process.