---
layout: inner
title: 'Geometric Morphometrics demonstrate the shape of seal whiskers'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera/blob/master/img/ezgif.com-crop.gif?raw=true'
lead_text: 'Using computer vision technology to extract 2D seal whiskers' parameter base on the Geometric Morphometrics method'
---

# Geometric Parameter Extraction on 2D Seal Whiskers

## Overview 

Whiskers are important sensor for seal detecting vibrotactile information from the environment. Seal whiskers are demonstrated the diversity of shapes and most of them are exhibit a beaded morphology with repeating sequence of crests and troughs along. Currently it is unclear the diversity of shape affects environmental signal modulation. In order to get some clues of it, this project implement geometric morphometrics method to demonstrate the 2D shape of whiskers.   
 
The 2D whiskers graphs are in two kinds of background - green and black, as shown on figure 1 and figure 2. Rulers are placed next to whisker for the purpose of referencing the size of whiskers.  The goal for the project is to find out the parameters as shown on table 1. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/green_bg.png?raw=true" alt="drawing" height="400"/> </> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/back_bg.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 1: green and black background whisker photo</center>*


<!-- <p align="right"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/back_bg.png?raw=true" alt="drawing" width="200"/> </p>  

*<center>Figure 1: green background </center>*



![alt-text-1](https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/green_bg.png?raw=true "title-1") ![alt-text-2](img/whisker_peaks.png "title-2")![alt-text-3](img/whisker_peaks.png "title-3")  -->

<center>

| Parameters Name    | Explanation    |
| :------------- | :------------- |
|Whisker Length       | Length of Seal whisker |
|D_base | Base diameter   |
|D_tip   |Tip diameter   |
|Ratio_R   | D_base/D_tip   |
|CenterLine |All points position along the centerline |
|distance   |The straight line length from whisker tip to whisker base |
|crest positions | All crests positions on whiskers|
|through positions | All crests positions on whiskers|
</center>

*<center>Table 1: Parameters extracted from the whiskers </center>*

## Process

The process of geometric morphometrics of seal whiskers are as shown on the figure2. There are mainly four steps: get basic information from original graph from figure 1; build mask from the original image; Generate centerline from the mask, and calculate whisker parameters. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/process_flow_chart.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 2: Geometric Morphometrics Process Flow Chart </center>*

### Get basic Information 

The first step is to get basic whisker information from original graph. As shown on figure 2, the left graph is cropped from ruler in 10 mm, which would be used as a reference to estimate whisker's size. The right graph is a single whisker which one need be extracted its parameters. User also need to locate whisker's tip and base positions as shown on red circle part on the whisker. 





