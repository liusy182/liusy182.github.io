---
layout: post
title:  CNN case studies and practical advices
date:   2018-05-09 00:00:00 +0800
categories:
tags:
---

Case studies of famous CNN networks introduced in coursera
[Convolutional Neural Networks] given by Andrew Ng.

## LeNet - 5 (1998)

Trained on gray-scale images with dimension 32 x 32 x 1.

```
    32 x 32 x 1
        |
    6 filters,
    filter = 5 x 5,
    stride = 1
        |
    28 x 28 x 6
        |
    avg pooling,
    filter = 2 x 2,
    stride = 2
        |
    14 x 14 x 6
        |
    16 filters,
    filter = 5 x 5,
    stride = 1
        |
    10 x 10 x 16
        |
    avg pooling,
    filter = 2 x 2,
    stride = 2
        |
    5 x 5 x 16
        |
    fully connected layer,
    120 neurons
        |
    fully connected layer,
    84 neurons
        |
    y - 10 possible outputs

```

## AlexNet

```
    227 x 227 x 3
        |
    96 filters,
    filter = 11 x 11,
    stride = 4
        |
    55 x 55 x 96
        |
    max pooling,
    filter = 3 x 3,
    stride = 2
        |
    27 x 27 x 96
        |
    filter = 5 x 5,
    'same' convolution
        |
    27 x 27 x 256
        |
    max pooling,
    filter = 3 x 3,
    stride = 2
        |
    13 x 13 x 256
        |
    filter = 3 x 3,
    'same' convolution
        |
    13 x 13 x 384
        |
    filter = 3 x 3,
    'same' convolution
        |
    13 x 13 x 384
        |
    filter = 3 x 3,
    'same' convolution
        |
    13 x 13 x 256
        |
    max pooling,
    filter = 3 x 3,
    stride = 2
        |
    6 x 6 x 256
        |
    fully connected layer,
    686*256=9216 neurons
        |
    fully connected layer,
    4096 neurons
        |
    softmax,
    1000 possible outputs
```

## VGG 16

Instead of having so many hyperparameters, focus on conv-layer and use 'same'
convolution. This is a relatively large network with around 138 million
parameters.

```
    224 x 224 x 3
        |
    64 filters,
    'same' convolution
        |
     64 filters,
    'same' convolution
        |
    224 x 224 x 64
        |
    max pooling,
    filter = 2 x 2,
    stride = 2
        |
    112 x 112 x 64
        |
    128 filters,
    'same' convolution
        |
     128 filters,
    'same' convolution
        |
    112 x 112 x 128
        |
    max pooling,
    filter = 2 x 2,
    stride = 2
        |
    56 x 56 x 128
        |
       ...
        |
    7 x 7 x 512
        |
    fully connected
        |
    fully connected
        |
    softmax

```

## ResNets

Works very well for deep networks.

## Inception network

Try different filters and stack them together

```
               ...
                |
          28 x 28 x 192
    |       |       |       |
   1x1     3x3     5x5   max pool
   64   +  128  +   32  +   32
        'same' convolution
                |
          28 x 28 x 256
```

Inception network has problem of computational cost. An alternative solution is

```
               ...
                |
          28 x 28 x 192
                |
     1 x 1 'same' convolution
          16 filters
                |
          28 x 28 x 16
    |       |       |       |
   1x1     3x3     5x5   max pool
   64   +  128  +   32  +   32
        'same' convolution
                |
          28 x 28 x 256
```

Input / output dimensions are the same. We have reduced computation cost by
using **1 x 1 convolution** to reduce the size. (120m to 12.4m reduction)


[Convolutional Neural Networks]: https://www.coursera.org/learn/convolutional-neural-networks