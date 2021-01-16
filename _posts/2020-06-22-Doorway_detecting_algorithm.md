---
layout: inner
title: 'Automated Perception of Doorway Locations for Assistive Wheelchairs'
date: 2020-6-22 14:15:00
categories: development
type: project
tags: ROS Perception Camera-Sensor C++ 
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/title.png?raw=true'
lead_text: 'Using Computer vision for automated detection doorway of assistive shared-control Wheelchairs'
---

# Automated Perception of Doorway Locations for Assistive Wheelchairs

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

However, if we check the segmented inlier plane after first extracting iterations (as shown in figure 3), we find some interesting results.  The MLESAC is missing a corner but RANSAC is getting a full point cloud plane.  Those missing points on the MLESAC would accumulate to the following extracting iterations, which would potentially increase the overall extracting iterations running time. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RANSAC_MLESAC_IMAGE.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 3: MLESAC and RG segmentation after first extracting plane iterations; the white point clouds are the segmented plane after the algorithm’s first iteration, while the green marker is the detected door.
)</center>*


#### Tests Pt.2: comparison of RANSAC and MLESAC segmentation methods on the doorway assistance algorithm in the real world 
- It also compared the RANSAC and MLESAC in the real-world (as shown on table 2). At each position, we tested two different leaf_size values. 
    - Leaf_size is a parameter used on the voxel grid filter; larger leaf_size means a smaller point cloud size and a coarser point cloud resolution[4].  
- At the least noisy position (based on the camera property, intel is about 2.0 m), RANSAC and MLESAC have the same number of total segmented planes and detected doorway(s). 
- After increasing the leaf_size from 0.01 to 0.05, the segmented planes of MLESAC are larger than those of RANSAC. 
- When the camera’s view contains a large amount of visual noise/clutter,, more total segmented planes are found by MLESAC than by RANSAC, and sometimesMLESAC is unable to detect the doors. 
When we increase the leaf_size from 0.01 to 0.05 (akin to downsampling the point cloud’s size), neither algorithms can find the plane. 
  - NOTE: the total number of planes found by MLESAC is greater than those found by RANSAC. 
  - A greater total number of segmented planes means a greater number of iterations (wherein plane extraction takes place), which will result in greater overall running time.
  - In the real world, if the user can not detect the door immediately, it might cause some potential safety problems.  The data collected in these tests suggests that MLESAC has  found the most informative planes in the presence of a noisy real-world scene than RANSAC does (in the context of the doorway detection algorithm on which these methods were implemented). 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/table2.png?raw=true" alt="drawing"/> </p>  

*<center>Table 2: Comparing RANSAC and MLESAC in real-world (in Narrow-field of view without visual clutter and wider-field of view with visual clutter based on different leaf_size)
)</center>*

--- 

### New Algorithm - RG:
Algorithm Introduction: 
The next algorithm is implemented with Region-Growing (RG) segmentation. “RG algorithm is to merge points which are close enough in terms of the smoothness constraint (curvature and normal). Thereby, the output of this algorithm is the set of clusters, where each cluster is a set of points that are considered to be a part of the same smooth surface.” [5]

#### Tests Pt.3: Region_growing segmentation on simulation world

The Region-Growing segmentation is tested on the simulation world by placing the wheelchair at two positions: facing to door gap and offset angle to door gap. The region growing can successfully segment planes successfully (figure 3). 


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 3: Region_Growing segmentation on simulation world. </center>*

#### Tests Pt.4: Region_growing segmentation on real - world

Region-Growing is tested in the real world where the distance between camera and door is smaller than 2.0 m (based on the intel camera property, point cloud would become wavy if the distance is greater than 2.0 m), and all planes are detected successfully (figure 4). Those red points do not belong to any clusters, since the number of those points are smaller than the threshold of the minimum number of points clusters. However, because of the high number of clusters, the original doorway detection algorithm is not working anymore. Since region-growing is finding each plane separately instead of finding the door gap from two parallel planes as doorway detections’s original algorithm. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG_Real_world.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 4: Region_Growing segmentation test on the real - world (distance < 2m)</center>*

#### Algorithm Steps:  
Therefore, we need to design new algorithms for doorway detection which implements Region-Growing  (the brief outline of the algorithms as shown on figure 5). 

- Step1: After point clouds are subscribed, it would be passed to a passthrough filter to remove the ground plane by getting rid of points whose z values are smaller than 0.05 of the max_z of overall point cloud. It is safe to remove the ground plane since we only care about wall planes on the doorway detection algorithm. Also this step can sampling down the point cloud size, and increase the running efficiency for the following steps. 
- Step 2: The next step is using Region - Growing to cluster planes. Some parameters need to be tuned in order to segment planes correctly.
  - If the object surface are not smooth,  the parameter of SetSmoothnessThreshold and setCurvatureThreshold need to be set as larger number
  - If the size of the object plane is large, then the setMinClusterSize needs to be tuned to a larger value.
  - To use more accurate normal values in the Region-Growing function, the setKsearch parameter on the normal_Estimation function needs to be tuned to larger. However, it might run into time efficiency problems if the setKsearch value is too large.
- Step 3:  Each clustered plane needs to be checked if the plane's height is greater than 1m (ADA standardized height), if so the plane would be added into potential_wall_list. 
- Step 4: Then for each plane in the potential wall list, we will get two endpoints points (T1,T2) from each cluster (projected into the x-y plane) which corresponds to the min_x, max_x values, and add two endpoints’ x and y value (T1.x, T1.y, T2.x, T2.y) into the Endpoint_list. 
- Step 5: The next step sort (IntroSort) [9] the Endpoint_list based on T1.x values. 
- Step 6:  The last step would be check the gap distance for every two planes, and a doorway would be detected if the gap distance is within the ADA standardized door width (36 inches to 48 inches).  

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG_algorithm.png?raw=true" alt="drawing" height="500"/> </p>  

*<center>Figure 5: New Doorway assistance algorithm implement with RG</center>*

#### Tests Pt.5: New doorway detection algorithm with RG algorithm are tested on simulation 
The new doorway detection algorithms with RG segmentation are tested on simulation (as shown on figure 6). Wheelchair is placed at two positions: facing to the door gap, and offset angle to wall gap. Both positions can detect the door successfully.   

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG_simulation.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 6: Doorway detection algorithm with RG algorithm tested on simulation; The white part is the subscribed point clouds, the green marker is detected doorway, and the yellow plane is one of the segment clusters after the Region-Growing segmentation.</center>*

#### Tests Pt.6: New doorway detection algorithm with RG algorithm tested on the real - world

Doorway detection with RG algorithm is tested on real-world (as shown on figure 7), where the distance between camera and doorway is smaller than 2 m. The algorithm can detect the doorway successfully. The yellow points is one of the endpoints of a segment plane from RG. 

However, when the distance between camera and doorway is greater than 2m, the point clouds become very wavy. Since the camera has better performance when it's distance to the object is under 2m based on the intel datasets[6]. When camera is about 3.5 m away from the doorway  (as shown on figure 8), the points clouds are wavy and segmented out 27 clusters from the Region-Growing, we tried to solve it by turning the parameter of normal threshold to a larger value (form 3 degree to 4 degrees), but it cause the part of ceiling and the ground on the same plane as the left wall. By solving this problem in the future, we can try to use different cameras or applied filters to get rid of wavy noise. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/Doorway_RG.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 7: Doorway detection algorithm with RG algorithm tested on simulation; The white part is the subscribed point clouds, the green marker is detected doorway, and the yellow plane is one of the segment clusters after the Region-Growing segmentation.</center>*


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/RG_REAL_WORLD.png?raw=true" alt="drawing" height="400"/> </p>  

*<center>Figure 8: Doorway detection algorithm with RG algorithm tested on simulation; The white part is the subscribed point clouds, the green marker is detected doorway, and the yellow plane is one of the segment clusters after the Region-Growing segmentation.</center>*

#### Tests Pt.7 and Pt.8: Door detection with RG algorithm on real-world
In order to compare the accuracy of doorway detection between RG and RANSAC algorithm, we did distance test and angle test. For the distance test, a wheelchair is placed in front of the door gap, where the distance is from 1.2 m to 4.8 m, and interval range is 0.2m (test 7) or 0.1 m(test 8); For the Angle test, a wheelchair is placed at 2.4 m distance away from the door gap but with different angles, the angle range is from 30 - 150 degrees and the interval range is 10 degrees (test 7) or 5 degrees(test 8) .  

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/test_set_up.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 9: test pt.7 and test pt.8 setup.</center>*




#### Time improvement:
The running time of Normalized_Estimation function is about half of the overall running time of the new doorway assistance region growing algorithm. To improve the running efficiency of Normalized_Estimation function, we increased the value of leaf_size parameter in the voxel grid filter functions, and decreased the setKsearch parameter on the Normal_Estimation function. Both ways can efficiently reduce the Normal_Estimation function’s running time (as shown on Table 3).   

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/table3.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 9: test pt.7 and test pt.8 setup.</center>*

#### Tests Pt.7 Angle and distance test of door detection with RG algorithm on real- world (leaf_size = 0.01)

- On the Distance_Test - Door_Position results (Figure 8), the door position error is increased along the linear displacement on the RG algorithm. One interesting thing we found is there are some extra columns (one or two) of point clouds on the upper height of two sides of walls. Those columns only have a small amount of height from instead of all the way lengthened into the ground. Each wall has different performance of those column points (the amount can be different).  Since the doorway detection with RG algorithm is highly dependent on the extreme endpoints of the whole wall, some extreme edge points might not be able to be detected until the wheelchair is moving far away from the wall gap so the camera can see a greater view scope. 
- However,  the camera detection properties may not change perfectly linearly along the distance displacement, at some certain positions where the camera might miss some points, which might be the extreme points of the wall even if the camera is moved far away. So this sensor property could cause fluctuations, but it doesn't affect the overall increasing path of RG along the distance displacement. All above properties can also explain the door gap width error path on RG which is an increased but fluctuated (Figure 10).
- The collecting points on the RANSAC segment parallel wall plane would change while the wheelchair is moving away from the door gap and the camera has a greater view scope (based on the RANSAC segmentation properties). The edge points on the stripe might be collected differently, so the door gap width error would have a fluctuate path (Figure 9). 
- The fluctuation range of the door position error on the 3D graph of RANSAC (figure 8)  is close to the door gap width error’s change range (figure 10) . The 2D graph the door position error on the RANSAC algorithm is placed on a large scope (figure 8), so it would look more constant than the path shows on figure 9. 
- Comparing the RANSAC and RG’s  door position error on the distance test, the RG’s  has greater slope and is more fluctuate than RANSAC’s . They have a crossover point at 2.0 m.  Before 2.0 m, the RG’s error value is smaller than RANSAC’s, but after 2.0 meter, the RANSAC’s error is greater.
- On the door gap width error of distance test (figure 9). RG’s error is increasing along the distance, and RANSAC’s error path is more fluctuated than RG’s .  

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/1.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 10: distance_test results of door position error (leaf_size = 0.01)</center>*

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/2.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 11: distance_test result of door gap width error  (leaf_size = 0.01)</center>*

- On the Angle_Test -- Door_Gap_Width result (figure 12), the door gap width error is fluctuated greatly on the RANSAC algorithm. When the wheelchair moves to the edge angle, the segmented parallel plane from RANSAC would be affected by the thickness of the wall. It would be inclined instead of keeping exactly parallel as the real wall plane surface (We did another test on two non-parallel walls, the RANSAC algorithm can segment out a parallel plane).  So the collected points on the strips would be different and might miss some important edge points, which would cause the fluctuating error path of door gap width on RANSAC (figure 12) when the wheelchair is placed at the offset angle, and the  small error value at 90 degrees where wheelchair is facing in front of the door. However, the door position error path on the RANSAC is pretty constant (figure 13), since the stripe is symmetric to the center axis of the door gap even if  the wheelchair moves to an offset angle to the door gap.  
- The RG’s door_gap_width error is smaller at center, and larger at side angles (figure 12). Since when the wheelchair moves to the offset angle, the normal and curvature values around the edge would change and cooperate with the points on the side surface of the wall, it would cause the more points around edge points not included into the wall plane. However, since not all points' normal values are exactly the same along the whole column line, sometimes there are few points included in the wall plane but not all most other points. Those special points might become the door start/end points on the doorway algorithm, which would cause the error of door gap width and door position fluctuated. 
- The distance between wheelchair and door gap setup on the angle_Test is 2.4 m, the error value of door position on the angle test at 90 degrees (figure 13) is similar to the distance test’s error at 2.4 meters (figure 10). On the distance test, the smallest error value of door position on RG is at 1.2 m, so may be a good idea to do the angle test with a radius of 1.2 meters in the future. 

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/3.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 12: angle_test results of door gap width error (leaf_size = 0.01)</center>*

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/4.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 13:  angle_test results of door position error (leaf_size = 0.01)</center>*

#### Tests Pt.8: Angle and distance test of door detection with RG algorithm on real- world (leaf_size = 0.05) 

While increasing the leaf_size from 0.01 to 0.05 to downsample the point cloud size, the door gap error on angle and displacement tests are fluctuating greater on both RG and RANSAC algorithms (figure 15 and 16). The distance between each point and the total amount of points would be increased when the leaf_size is changed from 0.01 to 0.05. So on RANSAC segmentation, the amount of points numbers are smaller on the stripe and each point would count more weight on the stripe after we increased the leaf_size, and the point's position on the stripe edge would greatly affect the door gap width error. The RG has a smaller fluctuation effect than RANSAC’s, because the RG gets the door start/end point from the whole plane instead of a small point cloud size of the stripe. Changing the leaf size doesn’t affect the error path shape of door position on RG and RANSAC on both angle and displacement tests. 


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/5.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 15: distance_test result for door position (leaf_size = 0.05)</center>*

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/6.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 16:distance_test results for door gap width(leaf_size = 0.05)</center>*


<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/7.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 17: angel_test results for door gap width (leaf_size = 0.05)</center>*

<p align="middle"> <img src="https://github.com/luxi-huang/portfolio/blob/master/img/posts/doorway_detection/8.png?raw=true" alt="drawing"/> </p>  

*<center>Figure 18: angle_test results for door position(leaf_size = 0.05)</center>*

---
## Conclusion 
The door thickness could cause the fluctuation error path of the door position on the RG and RANSAC while wheelchair placed at offset angle. The RG’s error (door position and door gap width) have an increasing pattern along the distance test. The segment plane from the RANSAC would be changed on when the wheelchair moving away from the door gap or turning at offset angle, but the center of door position is not changed much as door gaps. Increasing the leaf_size would cause a greater fluctuation path of the door gap width error on displacement and angle tests for both algorithms (RANSAC and RG), and it would affect greatly on RANSAC than RG. However, increasing leaf_size slows down the running time dramatically on RG but not much on the RANSAC, therefore RG is more suitable to use greater leaf_size value than RANSAC.  

---
## Future Scope
- Next step for the doorway assistance project can test different angles on two collinear walls, and find the relationship between the angle size and segmentation plane shape of RG and RANSAC algorithms. 
- Do the Angle test with radius at 1.2 meters instead of 2.4 meters of RANSAC and RG algorithm, since RG has best door position error performance at 1.2 m.  
Also we can do the angle and distance test for RG and RANSAC  in the real-world, and  determine - crossover conditions for combined algorithms. 
- Misc. book-keeping/ease of use for the code that we didn’t have time to get to; e.g. including additional flags for testing different parameters or switch algorithms without rebuilding the code.



---
## Reference

[1] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/14iros_jain.pdf

[2] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/13icra_derry.pdf

[3]http://www.robots.ox.ac.uk/~vgg/publications/papers/torr00.pdf

[4] http://www.robots.ox.ac.uk/~vgg/publications/papers/torr00.pdf

[5]https://pcl.readthedocs.io/projects/tutorials/en/latest/region_growing_segmentation.html#region-growing-segmentation

[6] https://pcl.readthedocs.io/projects/tutorials/en/latest/voxel_grid.html#voxelgrid

---