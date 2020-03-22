---
layout: inner
title: 'MegaBlocks'
date: 2020-01-18 14:15:00
categories: development
type: project
tags: ROS BaxterRobot Path-Planning Robot-Control
featured_image: '/img/posts/KF/ezgif.com-gif-maker-2.gif'
lead_text: 'This Project applied BaxterRobot to build Lego pyrimid by using ROS '
---


<!-- https://github.com/luxi-huang/final-project-megabloks -->


# MegaBlocks Building LEGO Pyrimid

For this project, Baxter assembles a MEGA BLOKS pyramid from the blocks provided by the user. The flow is:
1. Run setup file to get Baxter into position
2. Human places blocks on tray
3. Baxter detects location of a block (with right hand)
4. Goal block is displayed to user on screen
5. Baxter navigates to block, picks it up, and places it in the pyramid location (with left hand)
6. Baxter repeats steps 3 and 4 until the pyramid is complete


![Hierarchy](/img/posts/KF/ezgif.com-gif-maker-2.gif)*<center>Figure 2: Senosr fuse for Depth Camera D435i</center>*


If you want see more detailed on code for this project. please visit: [https://github.com/luxi-huang/
final-project-megabloks](https://github.com/luxi-huang/
final-project-megabloks ){:target="_blank"}

[^1]: Currently working on this project, I will keep updating this post based on the progress of the thesis.
[^2]: The cover picture is taken from [the repo of the project](<https://arxiv.org/pdf/1710.09767.pdf>){:target="_blank"}
