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
 
The whiskers images are in two kinds of background - green and black, as shown on figure 1 and figure 2. Rulers are placed next to whisker for the purpose of referencing the size of whiskers.  The goal of the project is to extract from seal whiskers parameters (as shown on table 1) from original image(figure 1). 

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

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/process_flow_chart.png?raw=true" alt="drawing" height="600"/> </p>  

*<center>Figure 2: Geometric Morphometrics Process Flow Chart </center>*

### 1. Get basic Information 

The first step is to get basic whisker information from original graph. As shown on figure 2, the left graph is cropped from ruler with 11 bars which is in 10 mm, which would be used as a reference to estimate whisker's size. Also user to pick which whisker they want to extracted its parameters, and locate whisker's tip and base positions as shown in red circle in the right graph. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/raw_ruler.png?raw=true" alt="drawing" height="400"/>  <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker.png?raw=true" alt="drawing" height="400"/> 



*<center>Figure 2: Basic information from original graph (Left graph is from 11 bar on ruler, right graph is a single whisker that user want to extract its parameters. and its base and tip positions) </center>*

The original image are in two different backgrounds: black and green. The programme on this project is designed to automatically distinguish those two background by checking the image shape. The black background image is belongs to gray scale type, which only has a single color channel， but the green background graph is belongs to rgb type, which has three color channels. So it is feasible to distinguish those two background by checking its number of color channels, then match mask building algorithm based on its background. 


### 2. Build Mask

#### Step 1: Build Initial Mask
To build an initial mask for a green background image, it would check every pixel's color value. If it is is within the background green color range, then that pixel's color value would change to 0. Otherwise, it would change to 1.  Thereby, that's how it builds the initial mask, as shown in figure 3 left side image. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_only_1.png?raw=true" alt="drawing" height="400"/>  <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_only_2.png?raw=true" alt="drawing" height="400"/> 


*<center>Figure 3: Build initial image(right) and improve image by removing its small objects) </center>*

#### Step2. Improve mask - Remove small objects from binary image
Since the background does not only contain pure green color (figure 1 left side graph), and it also exists some noise colors, the initial binary image has many noise objects beside whiskers. It implements an area filter method to cancel those small objects. If the object area is small than the threshold, then we would change its color to its opposite value. Therefore, we can improve the mask by removing small objects as shown on the right site image in figure 3. 

#### Step3 and Step4. Improve mask - build blurry image

If we look closer on the step two image (figure 4). There are some black objects inside of the whiskers and reach to the background. In Order to eliminate it. I blur whisker edges and remove small objects again to build final mask. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_step3_step4.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 4: improve mask from step 2 to step 4 </center>*

#### Check whisker edge resolution
To check the resolution of final mask. I extract its edge and placed on original graph. It provide the view to show the resolution of final mask is high enough to extract its parameters. 

### 3.Generate Centerline
#### Step1 & 2 : Build initial Centerline and smooth the centerline 

Next step is building initial center line from final mask, as shown on figure 5. It would start from base to tip along y axis, and it would find the most concentration point of white pixels on the line (the orange line on figure 5) which is parallel to x axis for all y values. The equation of finding the most concentration point as shown on left side on figure 5. Then it smooth the initial centerline to improve the centerline. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/Initial_Center_Liner.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 5: Initial Steps of building Centerline </center>*


#### Step3: improve the centerline 

After the step 2 of generate centerline. We would improve the centerline by getting every two nearby points on centerline, and find its linear equation. Then build a perpendicular line to that linear lines. Two white pixel edge points could be found on every perpendicular line. The new centerline points would be the middle points of two edge white pixel positions for every perpendicular lines. As shown on 6, the middle point in green color is more center than original center lines. Therefore, we could use this method to build a new centerline as shown on figure 6 right side.   

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/center_line_step3.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 6: Improve the centerline from step 2 to step 3 Centerline </center>*


### Calculate Parameters
### Tip/Base Diameters 

The figure 7 shows the sum of white pixel numbers on the perpendicular lines on the centerline from base to tip. The blue lines are the original white pixel numbers, the black lines is smoothed from blue line. Base on the convex and concave points, we can find the crest/though numbers. The tip diameter would be end point of the blue line, the base diameter would be end point of blue lines.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/Screenshot%20from%202020-11-14%2023-43-54.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 7: Sum of white pixel numbers along the centerline </center>*

### Crest/Though Positions 

As shown on figure 8, it calculate the white pixel number on left and right side of centerline, we can find the left and right side white pixel numbers are not same. It is a interesting results that some whiskers' crest/though positions are not symmetric.    

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/all_three_lines.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 8: the white pixel numbers on right/left and whole centerline</center>*

Base on the concave and convex point on the figure 8, we can get crest/though positions on the left and right side of centerline. We draw them back to the original graph of whiskers as shown on figure 9. The crest are in blue starts, and though on the right side is in green starts, the though on the left side is on blue starts.

<p align="middle"> <img src="
https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker_with_peaks.png?raw=true" alt="drawing" height="600"/> 

*<center>Figure 9: crest/though positions on original whiskers</center>*



