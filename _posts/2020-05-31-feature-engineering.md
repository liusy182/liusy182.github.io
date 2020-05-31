---
layout: post
title:  Common techniques in ML feature engineering
date:   2020-05-31 00:00:00 -0700
categories:
tags:
---

Data preprocessing and feature engineering are important steps to create quality
input for a machine learning model. Here are some of the common techniques. The 
original notebook [can be found in kaggle](https://www.kaggle.com/liusy182/kernel4d6eddf443)

# Categorical data

For categorical data, one common way is to do one-hot encoding. If there are too 
many distinct values in a column, we can also group values with smaller 
distributions into a single category like "Others". Another approach is to do 
hierachical encoding. For example, we can group zipcodes by country, region or 
city, depending on how specific we want to go.

<script src="https://gist.github.com/liusy182/8d934016df56f639136af96665c9a0ed.js"></script>

# Missing values

The most straightforward way of handling missing data is to drop rows 
\([df.dropna](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.dropna.html?highlight=dropna)\) that have missing values, however, this approach might be 
undesirable in cases when dropping rows result to siginificant reduction in data 
size (e.g. > 20%). The remaining dataset could also have **more bias** when 
the null-valued entries are skewed.

Another similar approach is to drop columns (`df.dropna(axis=1`) that have 
missing values. The disadvantage is that we may loose important information from 
those columns, therefore it leads to a underfitted model.

**Data imputation** replaces missing values with the attribute mean, median or 
some other estimated values. Before choosing a data imputation method, it is 
important to understand why the data is missing and what is the most appropriate 
replacement. A few tips:

  - is usually more appropriate to replace categorical data with **most frequently used value**.
  
  - median is preferred over mean when there are outliers that could skew the mean.

<script src="https://gist.github.com/liusy182/2d20b752dd463b64301eb6b42d37479c.js"></script>

# Outliers

Outliers could originate from data collection error, unusual probability event, 
unsual causal process and etc. Outliers could cause model overfitting since the 
probabilities could skew towards outliers. Outliers can be detected with either 
[z-score](https://en.wikipedia.org/wiki/Standard_score) or percentile. Outliers 
can be removed by either [trimming](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.mstats.trim.html) 
or [capping](https://docs.scipy.org/doc/scipy-0.13.0/reference/generated/scipy.stats.mstats.winsorize.html). 

# Scaling

Algorithms like gradient desent and kNN are sensitive to features being on 
different scales. Scaling help to bring all the features on the same scale. There 
are 2 common ways to perform scaling: **min-max scalling** and **mean/variance scaling**. 
mean/variance scaling smoothes the impact on outliers.

<script src="https://gist.github.com/liusy182/2934c5a32f42c8ac45100c341d9921ab.js"></script>

# Text data
​
Raw text data can be extremely messing given its unstructured nature. A few common techniques 
to prepare text data:
​
- basic preprocessing.
    ```
    # lowercase
    string.lower()
    
    # remove leading and tailing whitespaces
    string.strip()
    
    # replace punctuation with space
    re.compile('[%s]' % re.escape(string.punctuation)).sub(' ', text)
    
    # collapse multiple spaces into one
    re.compile(r"\s+").sub(" ", my_str).strip()
    ```
    
- stop word removal. e.g. remove "the", "a", "is".
​
- tf-idf. The more frequent a word occurs (e.g. "the"), the lower the weight it gets.
​
- **stemming**. e.g. worked, working -> work.
​
- spelling correctionn. e.g. opinoin -> opinion.
​
- synonym normalization. e.g. happiness, joy -> joy.
​

[NLTK](http://www.nltk.org/) is a very useful library for text data cleansing.

<script src="https://gist.github.com/liusy182/c98de50e67497a05f8944bc47f325499.js"></script>
