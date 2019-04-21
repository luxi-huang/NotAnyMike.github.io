---
layout: inner
position: right
title: 'Cross-Convolutional Neural Networks applied to Finance'
date: 2015-02-20 14:15:00
categories: development
type: related
tags: Movement-prediction DNN
featured_image: 'img/posts/crossconvnets/diagram.png'
lead_text: 'Cross-Convolutional Neural Networks applied to Finance (working on ...)'

---

# Cross-Convolutional Neural Networks applied to Finance

This was an old project and I would summary it like this:

This project was part of the search carried on in the Algorithmic and Combinatorial Research Group back in 2017, the main idea was to apply a model which usually was used to predict the next *possible* frame of a sequence, for example character animation. The model predicted **all the possible movements** given only one **static** frame. The main objective was to adapt that model to a dataset of orders from any stock. The important thing here is that both problems require predicting stochastic outcomes (possible movements of the character or the price) using only static images (only one frame or a clever representation of prices) in the original case. The composition of the animated object was the quality which can allow the model to predict the movement and in the case of the stock prices, it was all the past closed and not closed order from a certain stock.

The model used a combination between convnets, auto-encoders and cross-convolutional NN to understand and decode the image in different elements and predict the “logical” movement of each part. Then, using the single elements decoded from the image, reconstruct the new image with the movement already applied, some what similar to a capsule net.

The original model used images in which the movement was easy to predict, like characters in a 2D world or exercise videos. The hypothesis was that the orders book in for any stock, shared certain important characteristics which allowed this kind of model to extract useful patterns to predict the future behaviour of the stock being analysed.

![diagram](/site/img/posts/crossconvnets/diagram.png)