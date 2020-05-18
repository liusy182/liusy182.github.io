---
layout: post
title:  Structure Machine Learning Projects
date:   2018-05-07 00:00:00 +0800
categories:
tags:
---

Notes from coursera course [Structure Machine Learning Projects] given by Andrew Ng.

**Orthogonalization** is the technique to design different parameters
(a.k.a knobs) so that each parameter can be tuned easily to achieve good
performance.

Example of knobs:

- bigger network

- adam

- regularization

- bigger dev set, test set

- change cost function

etc.

Steps:

# 1. Set up your goal

## use a single number metric

different networks may give different precision and recall, instead of evaluating
a network with precision and recall, we can use F1 score to combine precision
and recall so that it is reduced to a single evaluation matrix.

```
F1 score = 2 / (1/p + 1/r)
```

## use satisfying and optimizing metric

e.g. choose a classifier that maximize accuracy (optimizing metric) and ensures
a running time of < 100ms (satisfying metric)


## train/dev/test set distributions

A normal process is to use training set to train a lot of different ideas, then
use dev set to evaluate performance of these ideas. Eventually one idea stands
out and we will evaluate this idea using test set.

It is recommended to have Dev and Test set coming from the same distributions.
E.g. same geographical distribution, age group distribution etc.

Choose a Dev / Test set to reflect data you expect to get in the future and
consider important to do well on.

## train/dev/test set size

**60% / 20% / 20%** was very reasonable split early on. Now given that we have
much larger data sets, it is much more reasonable to have more data in training set. E.g. given 1 million data, it is reasonable to have a split of
**98% / 1% / 1%**.

A general rule is to set your test set big enough so that to given high
confidence in the overall performance of your system.

## when to change dev/test sets and metrics.

Example, network A gives better accuracy than B but introduces pornographic
images. In this case network B is preferred in certain cases.

One way to tune this is to introduce higher weight in error function. E.g. assign a weight `w` of `10` to images that are porn and `1` that are
not porn.

# 2. Compare with Human-Level Performance

**Bayes optimized error** is the best possible error that no one can pass its
accuracy.

**Human level error** can be taken as a proxy for bayes error.

Example:

- Human level error: 1%

- Training error: 8%

- Dev error: 10%

This is a high bias problem because there is a huge gap between human level error and training error. (**avoidable bias**)

How to reduce avoidable bias

1. train longer, better optimization

1. train bigger model

1. find better NN architecture

Example:

- Human level error: 7.5%

- Training error: 8%

- Dev error: 10%

This is a high variance problem because the gap between training and dev error is larger. (**variance problem**)

How to reduce variance

1. regularization

1. more data

1. find better NN architecture

# 3. Error Analysis

**What to do with incorrectly labeled examples?**

DL algorithms are quite robust to random errors in the training set. It is OK
to leave them as they are in training set.

For dev / test set, it might make sense to manually fix incorrectly labeled
examples. Make sure to apply the same process so that dev / test sets continue
to come from the same distribution.

**How to start building a new system?**

1. set up dev/test set and metric

1. build initial system quickly

1. use bias/variance analysis and Error analysis to prioritize next steps

# 4. Mismatched Training / Dev Set

**training-dev set**: same distribution as training set, but not used for training.

Example:

- Training error: 1%

- Training-dev error: 9%

- Dev error: 10%

this is a variance problem because the network does not do well on the same
distribution of data.


Example:

- Training error: 1%

- Training-dev error: 1.5%

- Dev error: 10%

this is a data mismatch problem because whatever works on training distribution
does not work well on dev set.

**How to address data mismatch problem?**

- manual error analysis to understand the difference

- collect more data to make data similar. Artificial data synthesis. When using
Artificial data synthesis, be aware that it may only target a small subset of
a bigger data pool

# 5. Learning from Multiple Tasks

## Transfer learning

A NN network has already learned a task. Tune it to apply to a different task.
E.g. a network can identify cats, tune is to identify dogs.

A common approach is to re-initialize the last layer of the network with random
weights, and re-train the network. This speeds up the training since only the
last layer of the network is re-trained.

If the performance is not satisfying, we can re-train more layers up in the chain.

**When does transfer learning make sense?**

- Task A and task B have the same input x

- You have a lot more data for task A than task B

- Low level features from A would be help for learning B

## Multi-task learning

A simplified autonomous driving car needs to detect multiple features:

- Pedestrians

- Cars

- Red lights

- etc

This is one example of multitask learning.

Alternatively we can train 4 different networks instead of using one. The advantage of using a single network is that lower level features can be shared and re-used.

# 6. End-to-end Deep Learning

Pros:

- let the data speak

- less hand-designing of components needed

Cons:

- may need large amount of data

- excludes potentially useful hand-designed components


[Structure Machine Learning Projects]: https://www.coursera.org/learn/machine-learning-projects/home