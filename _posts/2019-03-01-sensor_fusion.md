---
layout: inner
title: 'Mapping by Sensor Fusion with IMU and Camera (RGBD and Fisheyes)'
date: 2020-03-03 14:15:00
categories: development
type: project
tags: Vision-slam Sensor-fusion Computer-vision
featured_image: 'https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera/blob/master/img/ezgif.com-crop.gif?raw=true'
lead_text: 'Building mapping function on Intel realsense traking camera T265 and depth camera D435i individually, then compare their mapping qualities.'
---


<!-- https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera/blob/master/img/mapping.png?raw=true' -->

# Mapping by Sensor Fusion with IMU and Camera (RGBD and Fisheyes)


### Overview:
<!-- This is my MSc Thesis / University of Edinburgh[^1][^2]. Here is a short video: -->

In general, this project builds mapping functions on Intel realsense tracking camera T265 and depth camera D435i individually, then compares their mapping qualities. To achieve the mapping function on the depth camera D435i, the depth information from its two RGBD eyes could be fused with IMU data by applying the EKF filter. For the tracking camera T265, two fisheyes could perform as stereo cameras to measure the depth, then combine with its tracking properties to build a map.



### motivation:

The depth camera D435i has two RGBD eyes which can measure the depth but not the odom position. In contrast, the tracking camera T265 has two fisheyes, however, Intel only enables its tracking properties without a depth measurement function. As shown on figure 1, the wider view and visible light could provide a better performance on the tracking function. Perhaps that’s the reason Intel chose fisheyes on the tracking camera, and why they chose RGBD eyes on the depth camera, due to its narrow view and infrared light spectrum. Although Intel separated the tracking and depth functions to two different cameras, each camera contains an IMU and two eyes, which makes it possible to combine the tracking and depth functions together into one camera.

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/comparing_map_traking.png?raw=true)*<center>Figure 1: Trade off Traking and Depth Camera</center>*

---

### Building Map on Realsense Depth Camera D435i

Intel realsense cameras could access ROS with open sources. Figure 2 provides the sensor fusion process of mapping on depth camera D435i. Next, one implements the Madgwick filter on the raw IMU data to decrease the noise and fused data of the IMU. Then, accessing two RGBD eyes on RTabMap creates a cloud point and raw depth value. Next, IMU data and RGBD data should be fused by EKF to get a more accurate odometry position and mapping graph.

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/Sensor_fusion_D435i.png?raw=true)*<center>Figure 2: Senosr fuse for Depth Camera D435i</center>*

The figure 3 shows loop closure measurement on depth camera D4351. The RTabMap library comes with loop closure and ICP (integrated cloud points) function, which improves its mapping quality to be more accurate and efficient.


![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/ezgif.com-gif-maker-1.gif?raw=true)*<center>Figure 3: Loop closeure for Depth Camera D435i</center>*



<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/FVJkvy9j-2g" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->

---
### Building Map on Realsense Tracking Camera T265

The tracing camera T265 only provides its odometry, which does not have a function of depth measurement. However, if we rectify two fisheyes and treat them as stereo cameras by using openCV, we can get depth measurement and its graph as shown on figure 4.


![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/depth.png?raw=true)*<center>Figure 4: Depth graph from T265</center>*


If one treats two fisheyes on T265 as a stereo camera, then it is possible to apply the Rtabmap Stereo to build the cloud points. Instead of using the Madgwick filter to fuse IMU data like Depth camera D435i, the tracking camera offers its odometer values. One can fuse its odometer value and cloud point data by using RtabMap to create the map. As shown on figure 5, the map is built by implementing RtabMap loop closure and ICP.


![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/T265.png?raw=true)*<center>Figure 5: Depth graph from T265</center>*

---
### Comparing Results:

As discussed earlier, fisheyes could provide a wider view and more accurate odometry, but RGBD eyes offer higher quality on depth measurement. After one enables the tracking camera T265 to have a depth measurement function, and empower the depth camera D435i to have a tracking function, each camera can build a map by itself.


#### Compare Loop-closure:

Figure 6 compares the mapping quality and the reliability of mapping. Based on the information details and including the map, D435i has a better mapping quality than T265 when tested on a wide hallway. However, when tested on narrow hallways, which are smaller than 1.5m in width, the mapping was interrupted on the D435i camera, and only the T265 camera worked.

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/Mapping_camperision.png?raw=true)*<center>Figure 6: Comparing Maping T265 and D435i</center>*

D435i loses its odometry more easily than T265, and causes the mapping to be interrupted. Although the Rtabmap contains ICP (Integrated Cloud Points), the view size between RGBD eyes and fisheyes in T265 are different. Even when one adds the tracking function in the D435, it is still not as good as the T265. The narrowed hallway limited the ORB feature detection ability, and as a result, the loop closure would be hard to achieve on D435i.  

![Hierarchy](https://github.com/luxi-huang/portfolio/blob/master/img/posts/sensor_fusion/distance_compare.png?raw=true)*<center>Figure 6: Odom comparison at the end of loop closure </center>*

The Figure 7 compared  odometry when two camera complete loop closure. so we can find the at the end of the loop closure, the tracking camera T265 back to its start point closer than the depth camera D435i .  


#### Compare mapping odometry:
Since loop closure would fix the whole map as well as its odometry, I compare odometers’ accuracy of tracking campera and depth camera before they get loop closure. I hold each camera to walk 10 steps, and each step is $$ 0.3048 m (1 feet) $$, so I expect the first odometry is start from zero, and the new odometry would increase $$ 0.3048 m $$ than last one.If I label the odometry from camera as $$S_1,S_2,...,S_{10}$$, and label the expected odometry as $$ E_1,E_2,...,E_{10}$$. We can calculate their different, and to get mean $$m$$ and standard deviation $$Std$$:

$$
m = \frac{(S_1-E_1)+(S_2-E_2)+...+(S_{10}-E_{10})}{10}
\\
Std = \sqrt{\frac{(S_1-E_1)^2+(S_2-E_2)^2+...+(S_{10}-E_{10})^2}{10}}
$$

For T265 camera, we can find the get mean $$m = 0.001653$$, and  $$Std = 0.000548$$, but for D435i, the values are larger:  $$m = 0.003541$$, and  $$Std = 0.00492$$, so the odometry is more accurate on T265 than D435i.

---
### Conlusion:
After sensor fusing the IMU and camera data, the tracking camera T265 is able to detect depth, and the depth camera D435i would have tracking functions. One can build maps and do loop closure on each camera individually. The tracking camera T265 can find a more accurate odometer than D435i, but D435i’s point cloud map is more accurate and detailed than the tracking camera due to its featured RGBD eyes.

### Future Work:
Test two camera on the real robot, try the tasks like tracking or avoiding obstables, and compare which camera performance is better.


For code details of the project. please visit at my GitHub: [https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera](https://github.com/luxi-huang/Sensor-Fusion-Realsense-Camera){:target="_blank"}

---

[^1]: Currently working on this project, I will keep updating this post based on the progress of the thesis.
[^2]: The cover picture is taken from [the repo of the project](<https://arxiv.org/pdf/1710.09767.pdf>){:target="_blank"}
