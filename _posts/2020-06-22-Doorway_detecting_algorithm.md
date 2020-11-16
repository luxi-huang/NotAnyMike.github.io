---
layout: inner
title: 'Automated Perception of Doorway Locations for Assistive Wheelchairs'
date: 2020-6-22 14:15:00
categories: development
type: project
tags: ROS Perception Camera-Sensor C++ 
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/title.png?raw=true'
lead_text: 'Automated Doorway Detection for Assistive Shared-Control Wheelchairs'
---

# Automated Doorway Detection for Assistive Wheelchairs

- Skills: Perception, ROS, C++, RGBD-Camera Sensor, point-cloud 

---

## Introduction 

### Motivation 

Powered wheelchairs provide a mobility solution for people who unable  to  operate  a  manual  wheelchair,  for  reasons  of strength  or  impairment.  However,  operation  of  a  powered wheelchair can still be a difficult, tedious or challenging task. It not only because of limitations in  their  own  motor  control,  but  also  because  of  limitations  in the control interfaces available to them. [1] So it would be helpful if the operator can accurately sense their environment and locate goals. 

### Overview
This project build an automatic doorway (open door positions) detection system for wheelchairs with camera sensor.  This projects demonstrate the two new algorithms which were implemented on the original doorway detection algorithms[2], which are MLESAC (Maximum Likelihood Estimation Sample Consensus) segmentation and Region-Growing (RG) in the context of doorway assistance, including results from the tests of those both algorithms in the simulation and real-world. 

---
## Camera Filters -- Point Cloud Library Nodelets  
<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/camera_filter.png?raw=true" alt="drawing"/> </p>  


*<center>Figure 1: Camera Filters - Point Cloud Library Nodelets</center>*

Two RGBD cameras sensors are placed on two side arm of wheelchairs. each camera would pass through point clouds library library nodelets as shown on figure 1. 

- Pass through filter: this filter would convert camera axis and pass through distance within threshold 
- Cloud_throttle:  update the camera input frequency
- Voxel_grid: reduce the point cloud size but not change the point cloud shape

After left and right camera pass the voxel_grid, it would combine the viewpoints by using Combine point_cloud filter. Then the camera point clouds would be ready to pass into Doorway Assistance. 

---

## Algorithms 

### Original algorithm introduction (RANSAC)

<!-- <strong>bold text</strong>. -->

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/original_algorithm.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 2: Original door detection algorithm</center>*


 - Doorway detection’s original algorithm implements RANSAC (RANdom SAmpling Consensus) segmentation to fit a point cloud to a parallel plane model.On that plane all points which are at ADA standardized height(1m) stripe_height will be collected and be treated as a stripe.
   - The stripe (and plane) is a subset of the larger point cloud  

- Next, the algorithm performs an octree search from one side of the stripe (the side which contains the minimum X value) within a thresholded radius along the stripe to check if there exist any points within that radius. 
    - If so, the algorithm keeps searching until 0 neighbors can be found within that radius, and then treats that position as a door_start_point.
    - Then the algorithm will keep searching until some neighbor point(s) can be found within the search radius’s threshold.
    - If the search finds neighbor points, they will be marked as the door_end point. 

- Once the door_start and door_end points are found, the door gap width can be calculated (door gap width is defined as the distance between the door_start and door_end points). 
    - If the door gap width is within ADA standards (the range from 32 inches to 48 inches), then this gap would be a door gap[2].   

---

### New Algorithm - MLESAC:

- After the doorway detection pipeline subscribes to point cloud data, in both RANSAC and MLESAC segmentation methods it will go through the collection of inlier points.
    - Inlier points are defined as points which are considered within the error bounds while fitting the chosen model (in the case of Doorway Detection, the chosen model is a parallel plane)
- Next, those points are  extracted from the original point cloud and stored as a subset ‘inlier plane’.
    - The rest of the original point cloud part is considered the subsect of the outliers subset. 
- On the next iterations, the  respective segmentation method is applied to this subset of outlier points to segment out  new subsets of inlier and outlier points. 
    - Such iteration continues until the subsequent subset of outlier points contains fewer points than a predefined threshold on segmentation size [5]. 

#### Introduction to MLESAC:
 MLESAC is a later generation version of the RANSAC algorithm, it adopts the same sampling strategy as RANSAC to generate putative solutions, but chooses the solution to maximize the likelihood function rather than just the number of inliers. [3]

#### Tests Pt.1: comparing RANSAC and MLESAC on Simulation

Doorway_assistance of RANSAC and MLESAC are tested on simulations at three wheel chair positions: facing to wall, offset angle to door gap, and facing to door gap are shown on table 1. The world shown on the right picture is the general world we are using. The total segment plane means the number of plane extracting iterations for every subscribed point cloud. The results of MLESAC and RANSAC are the same based on the values on the table. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RANSAC_MLESAC_TABLE.png?raw=true" alt="drawing"/> </p>  

*<center>Table 1: Comparing RANSAC and MLESAC in Simulation (Gazebo)</center>*

However, if we check the segmented inlier plane after first extracting iterations (as shown in figure 1), we find some interesting results.  The MLESAC is missing a corner but RANSAC is getting a full point cloud plane.  Those missing points on the MLESAC would accumulate to the following extracting iterations, which would potentially increase the overall extracting iterations running time. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG_RANSAC_IMAGE.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 3: MLESAC and RG segmentation after first extracting plane iterations; the white point clouds are the segmented plane after the algorithm’s first iteration, while the green marker is the detected door.
)</center>*




---
## Reference

[1] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/14iros_jain.pdf

[2] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/13icra_derry.pdf

[3]http://www.robots.ox.ac.uk/~vgg/publications/papers/torr00.pdf