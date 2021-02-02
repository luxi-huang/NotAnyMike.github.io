---
layout: inner
title: 'Baxter Lego Builder'
date: 2020-01-18 14:15:00
categories: development
type: project
tags: ROS Python Manipulation Path-Planning Robot-Control
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/baxter/ezgif.com-gif-maker-2.gif?raw=true'
lead_text: 'This Project applied Rethink Robot Baxter to build Lego Pyramid by ROS '
---


<!-- https://github.com/luxi-huang/final-project-megabloks -->


# MegaBlocks Building LEGO Pyramid

- Skills:  ROS, Python, Path-planing, Inverse-kinematics, Robot-Arm-Control  

---

## Overview

This project uses ROS in python as the central platform. the Baxter robot assembles a LEGO mega blocks pyramid from the blocks provided by the human. The flow is as follow:

1. Human places LEGO blocks on the plate.
2. The LEGO blocks are detected on Baxter's camera image using OpenCV.
3. Among all detected blocks, one block is chosen at random and located in its 3D position.
4. The chosen block is displayed to users on the screen.
5. Baxter picks the blocks up and places them in the drop location using position control in MoveIt.
6. To double check, Baxter gripper hand push against the block to ensure that the block is pushed in. The gripper used force control in MoveIt.
7. Baxter repeats steps 3 to 6 until the pyramid is complete.

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/baxter/ezgif.com-gif-maker-2.gif?raw=true)*<center>Figure 2: Senosr fuse for Depth Camera D435i</center>*

View [full video](https://youtu.be/mz1FwBR94og)

---

## Human-Robot interaction
Before Baxter starts to assembles the pyramid, a human would place all Lego blocks on the plate. The last block the human places would be the color of the pyramid that Baxter will assemble. 

---

## Object Detection in computer vision
Baxter uses the camera on its right hand to detect all block locations, and targets one of the blocks that matches the human's desired color. The details includes following: 

#### Setup: 
Before Baxter detects the LEGO pyramid, Baxter need to locate the base plate. To do that, we placed several AR frames around the plate, and ensured AR frame needs to be under the camera's field of view.   

#### Process: 
we used a pre-calibrated intrinsic camera parameter to find an inverse projection ray corresponding to a pixel on Baxter's camera sensor. The 3D location corresponding to a pixel is calculated by finding the intersection of the inverse projected ray and ground plane, which is defined by one of the AR tags. The 3D point corresponding to the pixel is determined in multiple AR frames and then these points are converted to Baxter's world frame representation.

#### Results: 
  Consequently, we obtain multiple points in the world frame that correspond to the same pixel in Baxter's camera image but obtained using different AR frames as reference. The median of all these points is found and that median point is taken to be the true 3D location corresponding to the pixel in the image. 

#### Challenge: 
  Sometimes AR tags are affected by the lighting and the blocks 3D pose are skewed by the camera's oscillates. Even worst-case scenario, the blocks are not detected. To solve this issue, we use multiple AR tags to increases both the robustness and the precision of the system. We were able to locate points within +/- 1 cm accuracy through this process. 

---

## PickUp and Placing
After Baxter locates the LEGO block position, we then use MoveIt to control Baxter's arm to do pickUp on LEGO block and place it into the goal position. 

#### Path planning trajectory
The Baxter arm executes would pick up the LEGO block and placed it to the goal position with a designed path planning trajectory, which implements inverse kinematics from MoveIt.  The trajectory is composed of 7 steps:
1. Arm moves above the pickUp position. 
2. Arm moves down in the z-axis direction at the pickUp position.  
3. Gripper closes to hold the lego block.
4. Arm moves up in the z-axis direction. 
5. Arm moves in a straight line on the XY plane and stops at the position above the goal position. 
6. Arm move done in the z-axis to match at the goal position.
7. The gripper opens. 

#### Gripper open and close: 
- In this project, the gripper open and close are controlled by the MoveIt. 

#### Gripper force control:

All lego blocks are inserted into the lego board before the Baxter arm picks them up. Baxter's gripper need to add some force to pull the block from the board. Similarly, the grippers should have some force to push the lego into the goal position when the arm assembles it. 

To achieve the above task, I use MoveIt to add force on step 4 and step 7 on the path planning trajectory. 

#### Challenge：

Challenge 1: To assemble the Lego parts, their holes need to be matched very precisely. This requires the goal position error should be very small. 
     
Challenge 2: To insert lego holes into another lego block, the gripper has to add force to it. Also, we add an extra path after step 7 on the path planning trajectory. 
  1. Open gripper 
  2. Move up the alone z-axis 
  3. close gripper 
  4. Add force on the gripper and push done. 

Those extra paths can double-check lego is push into the plane.     

---

### My contribution:
- Designed Baxter robot Path-planing trajectory by applying Moveit and inverse kinematics modified joint velocity limits and scaling to meet desired performance.
- Programmed Baxter grippers to close and open to grab the Lego pyramid.
- Using force control in MoveIt to force on Baxter grippers.

If you want to see more details on the code for this project. please visit my GitHub: [https://github.com/luxi-huang/
final-project-megabloks](https://github.com/luxi-huang/
final-project-megabloks ){:target="_blank"}

[^1]: Currently working on this project, I will keep updating this post based on the progress of the thesis.
[^2]: The cover picture is taken from [the repo of the project](<https://arxiv.org/pdf/1710.09767.pdf>){:target="_blank"}