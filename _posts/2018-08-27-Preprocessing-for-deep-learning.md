---
bg: "chardons.jpg "
layout: post
date: 2018-08-27
mathjax: true
title: "Preprocessing for deep learning: from covariance matrix to image whitening"
crawlertitle: "Pre-processing for deep learning: from covariance matrix to image whitening"
categories: posts
tags: ['linear-algebra', 'python', 'numpy', 'deep-learning']
author: Yash Karwa
jupyter: https://github.com/hadrienj/Preprocessing-for-deep-learning
twitterImg: Preprocessing-for-deep-learning/rotation_small
excerpt: "The goal of this post/notebook is to go from the basics of data preprocessing to modern techniques used in deep learning. My point is that we can use code to better understand abstract mathematical notions! Thinking by coding! 💥"
excerpt-image: '<img src="../../assets/images/Preprocessing-for-deep-learning/rotation.png" width="500" alt="Rotation to decorrelate the data" title="The rotation can decorrelate the data.">
<em>The left plot shows correlated data. For instance, if you take a data point with a big $x$ value, chances are that $y$ will also be quite big. Now take all data points and do a rotation (maybe around 45 degrees counterclockwise): the new data (plotted on the right) is not correlated anymore.</em>'
---

A notebook version of this post can be found [here](https://github.com/hadrienj/Preprocessing-for-deep-learning) on Github.

The goal of this post/notebook is to go from the basics of data preprocessing to modern techniques used in deep learning. My point is that we can use code (Python/Numpy etc.) to better understand abstract mathematical notions! Thinking by coding! 💥

We will start with basic but very useful concepts in data science and machine learning/deep learning like variance and covariance matrix and we will go further to some preprocessing techniques used to feed images into neural networks. We will try to get more concrete insights using code to actually see what each equation is doing!

We call preprocessing all transformations on the raw data before it is fed to the machine learning or deep learning algorithm. For instance, training a convolutional neural network on raw images will probably lead to bad classification performances ([Pal & Sudeep, 2016](https://ieeexplore.ieee.org/document/7808140/)). The preprocessing is also important to speed up training (for instance, centering and scaling techniques, see [Lecun et al., 2012; see 4.3](http://yann.lecun.com/exdb/publis/pdf/lecun-98b.pdf)).


  



