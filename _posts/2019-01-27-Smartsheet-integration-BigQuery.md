---
bg: "smartsheet.png "
layout: post
date: 2019-01-27
mathjax: true
title: "Smartsheet Integration: Big Query"
crawlertitle: "Smartsheet Integration: Big Query"
categories: posts
tags: ['smartsheet', 'python', 'numpy', 'pandas']
author: Yash Karwa
jupyter: 
twitterImg: Preprocessing-for-deep-learning/rotation_small
excerpt: "The goal of this post/notebook is to show how to scrape data from smartsheet and integrate with Big Query, Google Cloud.

---

A notebook version of this post can be found [here](https://github.com/hadrienj/Preprocessing-for-deep-learning) on Github.

The goal of this post/notebook is to go from the basics of data preprocessing to modern techniques used in deep learning. My point is that we can use code (Python/Numpy etc.) to better understand abstract mathematical notions! Thinking by coding! 💥

We will start with basic but very useful concepts in data science and machine learning/deep learning like variance and covariance matrix and we will go further to some preprocessing techniques used to feed images into neural networks. We will try to get more concrete insights using code to actually see what each equation is doing!

We call preprocessing all transformations on the raw data before it is fed to the machine learning or deep learning algorithm. For instance, training a convolutional neural network on raw images will probably lead to bad classification performances ([Pal & Sudeep, 2016](https://ieeexplore.ieee.org/document/7808140/)). The preprocessing is also important to speed up training (for instance, centering and scaling techniques, see [Lecun et al., 2012; see 4.3](http://yann.lecun.com/exdb/publis/pdf/lecun-98b.pdf)).

Here is the syllabus of this tutorial:

1. **Background**: In the first part, we will get some reminders about variance and covariance and see how to generate and plot fake data to get a better understanding of these concepts.

2. **Preprocessing**: In the second part, we will see the basics of some preprocessing techniques that can be applied to any kind of data: mean normalization, standardisation and whitening.

3. **Whitening images**: In the third part, we will use the tools and concepts gained in 1. and 2. to do a special kind of whitening called Zero Component Analysis (ZCA). It can be used to preprocess images for deep learning. This part will be very practical and fun ☃️!

Feel free to fork the notebook. For instance, check the shapes of the matrices each time you have a doubt :)



  
