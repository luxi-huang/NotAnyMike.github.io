---
layout: inner
position: left
title: 'Cross-Convolutional Neural Networks applied to Finance'
date: 2017-09-01 14:15:00
categories: development
type: related
tags: Movement-prediction DNN
featured_image: 'img/posts/crossconvnets/diagram.png'
lead_text: 'Short-term Stochastic Motion Prediction to the Stock Market'


---

# Cross-Convolutional Neural Networks applied to Finance

This was an old project and I would summary it like this:

This project was part of the search carried on in the Algorithmic and Combinatorial Research Group back in 2017-18. The main idea was to apply a model for Short-term Stochastic Motion Prediction to the Stock Market (based on this [paper](http://visualdynamics.csail.mit.edu/){:target="_blank"}). The model predicted **all the possible movements** given only one **static** frame using two sub-models, a motion and an image encoder, the first one takes care of encoding the motion information, the later decodes the images in smaller static parts from which the original object if build again. Followed by their proposed network structure, called Cross Convolutional Networks.

![teaser](/img/posts/crossconvnets/teaser.jpg)

For example, in the image above, the next movement of the girl in the left can be whether moving the leg up or down. This architecture is able to model those small movements stochastically. The main objective was to adapt that model to a dataset of orders from any stock (that is all the negotiating orders while the markets are open, even if the order was not executed and remained opened) to be able to model the movements in the stock price. 

The model used a combination between convnets, auto-encoders and cross-convolutional NN to understand and decode the image in different elements and predict the “logical” movement of each part. Then, using the single elements decoded from the image, reconstruct the new image with the movement already applied, somewhat similar to a capsule net.

The original model used images in which the movement was easy to predict, like characters in a 2D world or exercise videos. The hypothesis was that the orders book in for any stock shared certain important characteristics which allowed this kind of model to extract useful patterns to predict the future behaviour of the stock being analysed.

![diagram](/img/posts/crossconvnets/diagram.png)
