---
layout: inner
title: 'Geometric Morphometrics (Using Computer Vision to Help Biologists Measure and Classify Seal Whiskers)'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker.gif?raw=true'
lead_text: 'Extract whisker parameters from image '
---

# Geometric Morphometrics demonstrate on Seal whiskers

- Skills:  Computer-Vision, Image-feature-extraction, Matlab 

---

## Overview 

Whiskers are an important sensor for seals to detect vibrotactile information from the environment. Seal whiskers demonstrate the diversity of shapes and most of them exhibit a beaded morphology with repeating sequences of crests and troughs. Currently, it is unclear if the diversity of shape affects environmental signal modulation. In order to investigate, this project implements a geometric morphometrics method to demonstrate the shape of whiskers.

The whiskers images are in two kinds of backgrounds - green and black, as shown on Figure 1. Rulers are placed next to whiskers for the purpose of referencing the size of whiskers. The goal of the project is to extract from seal whiskers parameters (as shown on Table 1) from the original image(Figure 1).


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/original_graph.png?raw=true" alt="drawing" height="600"/> </p>  

*<center>Figure 1: green and black background whisker photo</center>*


<!-- <center> -->

<!-- | Parameters Name    | Explanation    |
| ------------- | ------------- |
|Whisker Length       | Length of Seal whisker |
|D_base | Base diameter   |
|D_tip   |Tip diameter   |
|Ratio_R   | D_base/D_tip   |
|CenterLine |All points position along the centerline |
|distance   |The straight line length from whisker tip to whisker base |
|crest positions | All crests positions on whiskers|
|through positions | All crests positions on whiskers| -->

<!-- </center> -->
<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/table.png?raw=true" alt="drawing" height="600"/> </p>  

*<center>Table 1: Parameters extracted from the whiskers </center>*

---
## Process

The process of geometric morphometrics of seal whiskers are as shown in Figure 2. There are four main steps: get basic information from the original graph from Figure 1; build a mask from the original image; generate centerline from the mask; and calculate whiskers’ parameters.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/process_flow_chart.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 2: Geometric Morphometrics Process Flow Chart </center>*

---

### 1. Get basic Information 

The first step is to get basic whisker information from the original image. As shown in Figure 2, the left graph is a cropped image of a 10 mm ruler, which is used as a reference to estimate whisker size. Also users can pick which whisker they want to extract its parameters, and locate the whisker’s tip and base positions, as shown in the red circle in the right graph.


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/graph_ruler.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 3: Basic information from original graph (Left graph is from 11 bar on ruler, right graph is a single whisker that user want to extract its parameters. and its base and tip positions) </center>*

The original image is on two different backgrounds: black and green. The program on this project is designed to automatically distinguish those two backgrounds by checking the image’s shape. The black background image belongs to a grayscale type, which only has a single color channel, but the green background image belongs to a rgb type, which has three color channels. So it is possible to distinguish between those two backgrounds by checking the number of color channels, then matching the mask building algorithm based on its background.

---
### 2. Build Mask

#### Step 1: Build Initial Mask
To build an initial mask for a green background image, it checks every pixel’s color value. If it is within the background green color range, then that pixel’s color value would change to 0. Otherwise, it changes to 1. Thereby, it builds the initial mask, as shown in the left side of Figure 3.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask.png?raw=true" alt="drawing" height="600"/> </p>  


*<center>Figure 3: Build initial image(right) and improve image by removing its small objects) </center>*

#### Step2. Improve mask - Remove small objects from binary image
Since the background does not only contain pure green color (left side of Figure 1), there are also some particles in other colors.The initial binary image has many small noise objects besides whiskers. I implement an area filter method to cancel those small objects. If the object area is smaller than the threshold, then I would change its color to its opposite value. Therefore, we can improve the mask by removing small objects as shown on the right side image in Figure 3.

#### Step3 and Step4. Improve mask - build blurry image
If we look closer at the Step2 image (Figure 4), there are some black objects inside of the whiskers that reach to the background. In order to eliminate it, I blur whiskers’ edges and remove small objects again to build the final mask.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/mask_step3_step4.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 4: improve mask from step 2 to step 4 </center>*

#### Check whisker edge resolution
To check the resolution of the final mask, I extract its edge and placed it on the original image. It shows the resolution of the final mask is high enough to extract its parameters.

---

### 3. Generate Centerline
#### Step1 & 2 : Build initial Centerline and smooth the centerline 
The next step is building the initial center line from the final mask, as shown on Figure 5. It starts from base to tip along the y axis, and it would find the most concentrated point of white pixels on the line (the orange line on figure 5) which is parallel to the x axis for all y values. The equation for finding the most concentrated point is shown on the left side of Figure 5. Then, I smooth the initial centerline to improve it.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/Initial_Center_Liner.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 5: Initial Steps of building Centerline </center>*


#### Step3: improve the centerline 

After Step 2 of generating the centerline, I improve the centerline by getting every two nearby points on the centerline, and find their linear equation. Then I build a perpendicular line to that linear line. Two white pixel edge points could be found on every perpendicular line. The new centerline points are the middle points of two edge white pixel positions for every perpendicular line. As shown on Figure 6, the middle point in the green color is more centered than the original centerline. Therefore, we could use this method to build a new centerline as shown on the right side of Figure 6.  

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/center_line_step3.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 6: Improve the centerline from step 2 to step 3 Centerline </center>*

---
### 4. Calculate Parameters
#### Tip/Base Diameters 

Figure 7 shows the sum of white pixel numbers on the perpendicular lines of the centerline from base to tip. The blue lines are the original white pixel numbers, and the black lines are smoothed from blue lines. Based on the convex and concave points, we can find the crest/trough numbers. The tip diameter is the starting point of the blue line, and the base diameter is the ending point of blue lines.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/Screenshot%20from%202020-11-14%2023-43-54.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 7: Sum of white pixel numbers along the centerline </center>*

#### Crest/Though Positions 

As shown on Figure 8, I calculate the white pixel numbers on the left and right side of the centerline, we can find the left and right side white pixel numbers are not the same. It is an interesting result that some whiskers’ crest/trough positions are not symmetric.

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/all_three_lines.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 8: the white pixel numbers on right/left and whole centerline</center>*

Based on the concave and convex points on Figure 8, I can get crest/trough positions on the left and right side of the centerline. We draw them back to the original image of whiskers as shown on Figure 9. The crests are marked in blue stars, and troughs on the right side are marked in green stars, the troughs on the left side are in blue stars.

<p align="middle"> <img src="
https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker_with_peaks.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 9: crest/though positions on original whiskers</center>*  

#### Convert pixels number to millimeter 

I use the Figure 2 ruler image and place it on a two dimensional graph with pixel numbers. The star points are the middle positions of every bar. Since every two nearby star points are in 1 mm distance, I can convert units from pixel numbers to millimeters.

<p align="middle"> <img src="
https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/rulur_2d.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 10: place ruler in 2D pixel graph</center>*

#### Whisker length 

After I convert the unit from pixel to millimeters, I can place whisker’s centerline on 2 dimensional Cartier coordinate. I calculate the length of centerline, and the straight line length from base to tip. Then I rotate the whisker to make it start from the original position and 10 percent of its overall length is a line with the x axis.

<p align="middle"> <img src="
https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/Rotated_whiskers.png?raw=true" alt="drawing" height="600"/>  </p>  

*<center>Figure 11: place and rotate whiskers in cartier coordinate</center>*


---

### code: 
The code are published on github: 
- https://github.com/luxi-huang/Geometric-Whisker-Parameter-Extraction

