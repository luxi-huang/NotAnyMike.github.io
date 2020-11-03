---
layout: inner
title: 'Geometric Morphometrics demonstrate the shape of seal whiskers'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera/blob/master/img/ezgif.com-crop.gif?raw=true'
lead_text: 'Building mapping function on Intel realsense traking camera T265 and depth camera D435i individually, then compare their mapping qualities.'
---

# Geometric Parameter Extraction on 2D Seal Whiskers

## Overview 

Whiskers are important sensor for seal deteacting vibrotactile information from the environment. Seal whiskers are demonstrated the diversity of shapes and most of them are exhibit a beaded morphology with repeating sequnces of crests and throughs along. Currently it is unclear the diversity of shape affects environmental signal modulation. In order to get some clues of it, this project implement geometric morphometrics method to demonstrate the 2D shape of whiskers.   
 
The 2D whiskers graphs are in two kinds of backgroud - green and black, as shown on figure 1 and figure 2. Rulurs are placed next to whisker for the purpose of referencing the size of whiskers.  The goal for the project is to find out the parameters ask shown on table 1. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/green_bg.png?raw=true" alt="drawing" height="400"/> </> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/back_bg.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 1: green and black backgound whisker photo</center>*


<!-- <p align="right"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/back_bg.png?raw=true" alt="drawing" width="200"/> </p>  

*<center>Figure 1: green backgound</center>*



![alt-text-1](https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/green_bg.png?raw=true "title-1") ![alt-text-2](img/whisker_peaks.png "title-2")![alt-text-3](img/whisker_peaks.png "title-3")  -->


| Parameters Name    | Explanation    |
| :------------- | :------------- |
|Whisker Length       | Length of Seal whisker |
|D_base | Base diameter   |
|D_tip   |Tip diameter   |
|Ratio_R   | D_base/D_tip   |
|CenterLine |All points position alonge the centerline |
|distance   |The stright line length from whisker tip to whisker base |
|crest positions | All crests positions on whiskers|
|through positions | All crests positions on whiskers|


*<center>Table 1: Parameters extracted from the whiskers </center>*

Process
- insert a process graph 
- explainations + graphs 
