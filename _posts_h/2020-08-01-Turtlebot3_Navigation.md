---
layout: inner
title: 'EKF SLAM on Turtlebot3'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker.gif?raw=true'
lead_text: 'Extract whisker parameters from image '
---

# EKF SLAM on Turtlebot3
- skills:

---
## Overview 


---
## Robot Model:
A diff_drive robot is created with two wheels and a caser in front in `URDF` format, and it can viewable on `Rviz` (As shown on Figure ). All robot configurations are saved in `yaml` file. It implement  `joint_state_publisher` plugin to control robot wheels.

 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/master/img/robot_descreption.png?raw=true" alt="drawing" height = "300" /> </p>  

*<center>Figure : Diff dirve robot model </center>*


---

## Rectangular Path-Planing

Before implement with diff drive robot, I controlled a turtle to move in a rectangular path continuously (As shown on Figure) by sending linear and angular velocities. I used a state machine to transition between each leg of trajectory. 

 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/master/img/rect.gif?raw=true" alt="drawing" /> </p> 

*<center>Figure : Control a turtle move in a rectangular path </center>*

I predict turtle pose based on the sending velocity command (linear and angular velocities), 
The below chart is calculated error between predict turtle pose and real turtle position in x, y and theta.  The errors are smaller than absolute value of 0.1 meters.

 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/master/img/turtleRect_errorPlot.png?raw=true" alt="drawing" /> </p> 

 ---









