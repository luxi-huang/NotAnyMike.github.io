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

- Skills: Perception, ROS, C++, RGBD-Camera Sensor

---

## Introduction 

### Motivation 

Powered wheelchairs provide a mobility solution for people who unable  to  operate  a  manual  wheelchair,  for  reasons  of strength  or  impairment.  However,  operation  of  a  powered wheelchair can still be a difficult, tedious or challenging task. It not only because of limitations in  their  own  motor  control,  but  also  because  of  limitations  in the control interfaces available to them. [1] So it would be helpful if the operator can accurately sense their environment and locate goals. 

### Overview
This projects demonstrate the two new algorithms which were implemented on the original doorway detection algorithms[2], which are MLESAC (Maximum Likelihood Estimation Sample Consensus) segmentation and Region-Growing (RG) in the context of doorway assistance, including results from the tests of those both algorithms in the simulation and real-world. 

---

## Original algorithm introduction (RANSAC)

<!-- <strong>bold text</strong>. -->

 - Doorway detection’s original algorithm implements RANSAC (RANdom SAmpling Consensus) segmentation to fit a point cloud to a parallel plane model.On that plane all points which are at ADA standardized height(1m) stripe_height will be collected and be treated as a stripe.
   - The stripe (and plane) is a subset of the larger point cloud  

- Next, the algorithm performs an octree search from one side of the stripe (the side which contains the minimum X value) within a thresholded radius along the stripe to check if there exist any points within that radius. 
    - If so, the algorithm keeps searching until 0 neighbors can be found within that radius, and then treats that position as a door_start_point.
    - Then the algorithm will keep searching until some neighbor point(s) can be found within the search radius’s threshold.
    - If the search finds neighbor points, they will be marked as the door_end point. 

- Once the door_start and door_end points are found, the door gap width can be calculated (door gap width is defined as the distance between the door_start and door_end points). 
    - If the door gap width is within ADA standards (the range from 32 inches to 48 inches), then this gap would be a door gap[1]   




---
## Reference

[1] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/14iros_jain.pdf

[2] https://cpb-us-e1.wpmucdn.com/sites.northwestern.edu/dist/5/1812/files/2016/05/13icra_derry.pdf