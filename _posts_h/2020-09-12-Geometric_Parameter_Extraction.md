---
layout: inner
title: 'Geometric Morphometrics Demonstrate the Seal Whiskers Parameters from Images'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera/blob/master/img/ezgif.com-crop.gif?raw=true'
lead_text: 'Using computer vision technology to extract 2D seal whiskers' parameter base on the Geometric Morphometrics method'
---

# Geometric Morphometrics Demonstrate the Seal Whiskers Parameters from Images 

## Overview 

Whiskers are important sensor for seal detecting vibrotactile information from the environment. Seal whiskers are demonstrated the diversity of shapes and most of them are exhibit a beaded morphology with repeating sequence of crests and troughs along. Currently it is unclear the diversity of shape affects environmental signal modulation. In order to get some clues of it, this project implement geometric morphometrics method to demonstrate the shape of whiskers.   
 
The whiskers images are in two kinds of background - green and black, as shown on figure 1 and figure 2. Rulers are placed next to whisker for the purpose of referencing the size of whiskers.  The goal of the project is to extract 2D parameters from seal whiskers from as shown on table 1. 

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

### 1. Get basic Information 

The first step is to get basic whisker information from original graph. As shown on figure 2, the left graph is cropped from ruler with 11 bars which is in 10 mm, which would be used as a reference to estimate whisker's size. Also user to pick which whisker they want to extracted its parameters, and locate whisker's tip and base positions as shown in red circle in the right graph. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/raw_ruler.png?raw=true" alt="drawing" height="400"/>  <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker.png?raw=true" alt="drawing" height="400"/> 



*<center>Figure 2: Basic information from original graph (Left graph is from 11 bar on ruler, right graph is a single whisker that user want to extract its parameters. and its base and tip positions) </center>*



### 2. Build Mask


#### 2a. Check Background Color
The original graph has two different background: black and green. The programme for this project is designed to automatically distinguish those two background by checking the image shape. The black background image is belong to gray scale type, which only has one number of channel. However, the green background graph is belongs to rgb type, it has three color channels. So it is feasible to distinguish those two background by checking its number of color channels.

#### 2b. Build Binary Mask 
If the background is in green color, then we would change every pixel color value to 0 if it is in background green color range, and change its color value to 1 if it is not under background green color range. The result of initial binary mask as shown on figure 3 left side. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_only_1.png?raw=true" alt="drawing" height="400"/>  <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_only_2.png?raw=true" alt="drawing" height="400"/> 


*<center>Figure 3: Build initial image(right) and improve image by removing its small objects) </center>*

#### 2c. Improve mask - Remove small objects from binary image
since the background not only contains pure green color (figure 1 left side graph), it also has some noise colors. It cause the initial binary image has many noise objects beside whiskers. Base on the area of objects on the image, if the area is small than the threshold, then we would change its color to opposite value. The result as shown on figure 3 right site graph

#### 2d. Improve mask - build blurry image




