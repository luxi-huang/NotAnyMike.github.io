---
layout: inner
title: 'Baxter Lego Buider'
date: 2020-01-18 14:15:00
categories: development
type: project
tags: ROS python Rethink-Robot Path-Planning Robot-Control
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/baxter/ezgif.com-gif-maker-2.gif?raw=true'
lead_text: 'This Project applied Rethink Robot Baxter to build Lego Pyrimid by ROS '
---


<!-- https://github.com/luxi-huang/final-project-megabloks -->


# MegaBlocks Building LEGO Pyrimid

- Skills:  ROS, Python, Path-planing, Inverse-kinematics, Robot-Arm-Control  

---

### Overall

This project is using ROS in python as the central platform, Baxter assembles a LEGO mega blocks pyramid from the blocks provided by the user. The flow is:

1. Human places blocks on the plate.
2. Blocks tare detected on Baxter's camera image using OpenCV.
3. Among all detected blocks, a block is chosen at random and locate its 3D position.
4. Goal block is displayed to users on the screen.
5. Baxter picked the blocks up and places it in the drop location using position control in Moveit.
6. As a double check, the gripper hand is pushed against the block to ensure that the block is pushed in, which is applied by Force control in MoveIt.
6. Baxter repeats steps 3 and 6 until the pyramid is complete.

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/baxter/ezgif.com-gif-maker-2.gif?raw=true)*<center>Figure 2: Senosr fuse for Depth Camera D435i</center>*

View [full video](https://youtu.be/mz1FwBR94og)

### My contribution:
- Designed Baxter robot Path-planing trajectory by applying Moveit and inverse kinematics modified joint velocity limits and scaling to meet desired performance.
- Programmed Baxter grippers to close and open to grab the Lego pyramid.
- Using force control in Moveit to force on Baxter grippers.


If you want see more detailed on code for this project. please visit at my github: [https://github.com/luxi-huang/
final-project-megabloks](https://github.com/luxi-huang/
final-project-megabloks ){:target="_blank"}

[^1]: Currently working on this project, I will keep updating this post based on the progress of the thesis.
[^2]: The cover picture is taken from [the repo of the project](<https://arxiv.org/pdf/1710.09767.pdf>){:target="_blank"}
