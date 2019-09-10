---
layout: inner
title: 'What's next?'
date: 2019-05-01 14:15:00
categories: development
type: hidden
tags: ROS AV
featured_image: 'https://readwrite.com/wp-content/uploads/apple-self-driving-car.jpg'
lead_text: 'What is my next project?'

---

# What's next?

About my next project … I am considering building an Autonomous Car from zero. I can start with a very basic model, and move up and star impoving each module.

The project can be as follows:

The idea is to learn as much as posible about how things work in real life, so ROS is a mandatary tool,  which give us the possibility of writing c++ by default. A ROS project with several nodes would look something like this:

* One node taking care or launching the simulation and getting everything ready there. Once I have access to a better simulator or a real car, this node can be turned off or modified accordingly.
* A node for each sensor of the car, which will take care of reading and passing the information to the other modules. We can go big with the number and types of sensors, but I am fearly optimistic that in the future we will rely on cameras, so we can have a couple of cameras and IMU and maybe a GNS.
* A perception module, reading from cameras and publishing the position and types of objects. This would be one of the most exciting nodes, I want to use pseudo-lidar from obviouly monucular cameras to estimate a depth pixel-map, from there we can play around with different models to output the coordinates and type of objects.
* A map node reading from the perception and updating the estimation of the state of the car. Here we can use something simple as basic UKF.
* A very simple planer. As the map we don't want something very complicated. Here we can play around with path optimization algorithms to learn a bit more. The output will be a plan to follow.
* Controller, which will convert the plan into actions to the actuators. For example a PID controller will do. We can even ignore a low level controller for simplicity.

The interesting bit here is that, given that it is a simulation, every single node can be replace temporarly with the ground truth information, while I develop something more realistic.