---
layout: inner
title: 'Mapping and Object Detection by Whisker Sensor'
date: 2020-12-10 14:15:00
categories: development
type: project
tags: ROS Mapping Whisker-Sensor C++ 
featured_image: 'https://github.com/luxi-huang/Whisker_Robot/blob/master/img/all.gif?raw=true'
lead_text: 'Using Whisker sensor touching the objects and build map'
---

# Mapping and Object Detection by Whisker Sensor

- Skills: Mapping, C++, Object_detection, ROS

---

## Motivation  
Whisker robot could be play an important role in dark or other extreme environment, where other sensors like cameras might hardly contain its normal performance. Whisker robot can detect environments by using whiskers to touch with environments and help people finish some hard tasks.  

## Introduction 
This projects is building a map by using whisker sensors to detects the environment. It is using [Whisker Physics Simulator](https://github.com/SeNSE-lab/whiskitphysics) to build environment with obstacles and simulate whisker to detect the environment. Then, I transfer whisker sensor data to 2D scan data. After that, I use those data to implement with slam_toolbox.
 
 <p align="middle"> <img src="https://github.com/luxi-huang/Whisker_Robot/blob/master/img/Whisker_simulator.gif?raw=true" alt="drawing" /> </p>  

 *<center>Figure 1: Whisker Simulator </center>*


 <p align="middle"> <img src="https://github.com/luxi-huang/Whisker_Robot/blob/master/img/whisker.gif?raw=true" alt="drawing" /> </p>  

  *<center>Figure 1: Whisker environment detection </center>*

 <p align="middle"> <img src="https://github.com/luxi-huang/Whisker_Robot/blob/master/img/Map.png?raw=true" alt="drawing" /> </p>  

   *<center>Figure 1: Whisker environment detection </center>*

---

## Whisker simulation world 
As shown on Figure 2, the simulation world contains 5 cylinders. I controlled rat by sending it linear and angular velocities, and triggers the whisker moving back and forth. So the rat whisker can scan the environment, and output the whisker sensor data.  


## Whisker Sensor Data
Whisker is simulated to moving back and forth. Each whisker contains 20 segments, which can sense the whisker dynamics in force and moment when they touch with objects. Also whiskers can give feedback of which segment has collision with objects. Then they calculate the next whisker's motion path based on its force and moment. Also calculate current kinematics value in x, y and theta. 

After I get each segments kinematics data, I need to match each segments collision data with its kinematics data. After that, I am ready to estimate objects position.   

## Transfer whisker data to 2D scan data 
Whisker data are in 3D points. I need to transfer it to 2D laser-scan data, and use that data build a map. It order to do it, I transfer 3D points to 2D points by eliminate z axis value. Next I transfer 2D data to 2D laser-scan data by setting up ```scan_angle```, ```angle_increasement```, ```scan_time```, ```range_distance```.

## Map building
After I convert 3D whisker data to 2D scan data, I implement with slam toolbox to locate all obstacles in 2D map. 

### Future scope
-  Implement whisker mapping method into whisker robot in real world.
-  Implement with other object detection algorithm like circular detection algorithm.

[Github_pages](https://github.com/luxi-huang/Whisker_Robot)


