---
layout: post
title:  Utilities for exploratory data analysis
date:   2020-05-28 00:00:00 -0700
categories:
tags:
---

Exploratory data analysis is a very important initial step in a machine learning
workflow. It helps a ML practitioner to gain deeper understanding of the data, 
and to prepare for later steps in the workflow. This post documents a few useful 
data analysis utilities.

1. Quick explorations on the data columns, types, a few statistics and missing values.

    <script src="https://gist.github.com/liusy182/221792dc78f8db79d2a4183b43c8049f.js"></script>

2. Further, we can look at histograms to understand the patterns in each column.
 The bin size can be further increased (default is 10) so that to get more 
 granular data binning.

    <script src="https://gist.github.com/liusy182/2bbda4277c99e6a38f17a8852322da02.js"></script>

    <img src="{{ site.url }}/assets/tesla_hist.png">

3. We can also plot the pair-wise relations as a scatterplot matrix so that to
observe the correlations of the columns. I find this is very useful in observing
some initial patterns of the data.

    <script src="https://gist.github.com/liusy182/2b70d74152bad7a762916fc5dc6aefa2.js"></script>

    <img src="{{ site.url }}/assets/scatterplot.png">

More stuff can be found in this comprehensive [cheat sheet](http://www.utc.fr/~jlaforet/Suppl/python-cheatsheets.pdf)]
