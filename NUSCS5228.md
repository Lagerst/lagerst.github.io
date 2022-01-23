## Welcome

## [return to index](./index.html)

## Lecture 1 — Introduction & Overview

**What is Knowledge Discovery & Data Mining?**

data -> information -> knowledge -> wisdom

From Data to Knowledge:

data -> target data -> preprocessed data -> transformed data -> patterns -> knowledge

Not: Trivial extraction of information/patterns from data or Data analysis not yielding patterns

**Main goal: Generalizability**

Patterns should remain accurate over unseen data

Common causes: small and/or biased data

**Methods**

1. Association Rules
2. Clustering
3. Classification
4. Regression
5. Graph Mining
6. Recommender Systems

**Types of Attributes**

Nominal: Categorical(qualitative)
> Values are only labels
> 
> Operations: =, ≠
> 
> Examples: sex (m/f), eye color, zip code

Ordinal: Categorical(qualitative)
> Values are labels with a meaningful order
> 
> Operations: =, ≠, <, >
> 
> Examples: street numbers, education level

Interval: Numerical(quantitative)
> Values are measurements with a meaningful distance
> 
> Operations: =, ≠, <, >, +, -
> 
> Examples:  body temperature in ℃, calendar dates

Ratio: Numerical(quantitative)
> Values are measurements with a meaningful ratio
> 
> Operations: =, ≠, <, >, +, -, *, /
> 
> Examples: age, weight, income, blood pressure

**Types of Data Representations**

1. Noise
2. Outliers
3. Missing Values
4. Duplicated
   
**Exploratory Data Analysis (EDA)**

Identify Noise/Outliers

1. Histograms 直方图
2. Box plots 箱型图
3. Scatter plot 散布图

**Data Preprocessing**

1. Data Cleaning
2. Data Reduction
   1. Aggrrgation 聚合
   2. Binning and smoothing 分档 平滑
3. Data Transformation
   1. 消减
   2. Attribute construction
   3. Normalization 标准化，如 平均值·标准差 缩放
4. Data Discretization 离散化
   1. Converting continuous attributes into ordinal attributes. Example: Convert weight to a weight category
   2. Converting categorical attributes into numerical attributes. Example: Convert \{1, 2, 3\} to "\{\{0, 1\}, \{0, 1\}, \{0, 1\}\}"

## Lecture 2 — Clustering

**K-means**

1. Initialization: Select K points as initial centroids
2. Repeat until no change in assignments
    1. Assignment: assign each point to nearest cluster (i.e., centroid)
    2. Update: move each centroid to the average of its assigned points

> K-Means always converges!
> 
> Both assignment (A) and update (U) reduce SSE (or no changes)
> 
> Most improvement during the first iterations
> 
> Lloyd's algorithm returns a local optimum, not necessarily a global optimum

Limitations
1. K-Means is susceptible to "natural" clusters
2. Different initializations of centroids may yield different clusterings
3. Some initialization of centroids can yield empty clusters

Handling Empty Clusters*

**K-Means++ (K-Means Variants)**

Only changes initialization of centroids
(assignment/update steps remain the same)

Initialization process
1. Pick random point as first centroid
2. Repeat until K centroids have been picked
    1. For each point x, calculate distance to nearest existing centroid
    2. Pick random point for next centroid with
    probability proportional to distances

**X-Means (K-Means Variants)**

Automatic method to choose K

1. Run K-Means with K=2
2. Iteratively, run K-Means with K=2 over each subcluster
3. Split subcluster only if meaningful w.r.t. a scoring function

**K-Medoids (K-Means Variants)**

Restriction: centroids are chosen from the data points

**DBSCAN**

Basic characteristics
> Clusters: density-based
> 
> Clustering: partitional, exclusive, partial

define radius of a points neighborhood(r) and minimum number of points(MinPTs)

1. Find new cluster seed (core point), Repeat until x is a new label point
    1. Pick random unexplored point x
    2. If has less than MinPTs neighbors within r, label as noise (might change later)
2. Explore new cluster

DBSCAN always converges

DBSCAN is not completely deterministic
1. Noise and core points deterministic
2. Border points may be reachable from core points of different clusters

Limitations
1. DBSCAN cannot handle different densities
2. DBSCAN is generally very sensitive to parameters

How to Choose Parameter Values?
1. Informed by results of EDA
2. Density of data points has intuitive semantic meaning
3. Intuitive interpretations of meaningful parameter value

**K-means VS DBSCan**
> https://www.geeksforgeeks.org/difference-between-k-means-and-dbscan-clustering

K-means Clustering is more efficient for large datasets. DBSCan Clustering can not efficiently handle high dimensional datasets. 4. K-means Clustering does not work well with outliers and noisy datasets. DBScan clustering efficiently handles outliers and noisy datasets.

Kmeans is a least-squares optimization, whereas DBSCAN finds density-connected regions.Which technique is appropriate to use depends on your data and objectives. If you want to minimize least squares, use k-means. If you want to find density-connected regions use DBSCAN.