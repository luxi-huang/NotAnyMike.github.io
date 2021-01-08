---
layout: inner
title: 'EKF SLAM on Turtlebot3'
date: 2020-10-25 14:15:00
categories: development
type: project
tags: Computer-Vision  Image-Feature-Extraction MATLAB
featured_image: 'https://github.com/luxi-huang/portfolio/blob/master/img/posts/Whisker/whisker.gif?raw=true'
lead_text: 'Extract whisker parameters from image '
---

# EKF SLAM on Turtlebot3
- skills: C++, ROS, SLAM, Gazebo, Motion-planing. 

---
## Overview 


---
## Robot Model:
A diff_drive robot is created with two wheels and a caser in front in `URDF` format, and it can viewable on `Rviz` (As shown on Figure ). All robot configurations are saved in `yaml` file. It implement  `joint_state_publisher` plugin to control robot wheels.

 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/master/img/robot_descreption.png?raw=true" alt="drawing" height = "300" /> </p>  

*<center>Figure : Diff dirve robot model </center>*


---
## Rigid 2D library:
This is a totally scratch project, and it is started from building Rigid2d library, which containing 2D Lie Group operations for Transforms, Vectors and Twists as well as differential drive robot kinematics for odometry updates. The theory is based on [Modern Robotics](http://hades.mech.northwestern.edu/index.php/Modern_Robotics) by Kevin Lynch.


### 2D Lie Group library: 
It include  $SO(2)$ and $SE(2)$ calculations for transformation, vector and twists.  

### Differential Drive robot:
This class track the state of a differential drive robot as its wheel positions as updated. The core functions in this class are `twist to wheels ` and `wheels to twists`

- #### Twist to Wheels Speed
As shown on Figure of Robot model, the body length is $D$, and wheel radius is $r$.

The adjoints from body center to left and right wheels can be:
$$
A_{bl} = 
\begin{bmatrix}
1 & 0 & 0\\
-D & 1 & 0\\
0 & 0 & 1\\
\end{bmatrix},  

A_{br} = 
\begin{bmatrix}
1 & 0 & 0\\
D & 1 & 0\\
0 & 0 & 1\\
\end{bmatrix}
$$

So we can convert twist to velocity with equation of $V = A_{ib} \times V_b$

$$
V_l = 
\begin{bmatrix}
1 & 0 & 0\\
-D & 1 & 0\\
0 & 0 & 1\\
\end{bmatrix}

\begin{bmatrix}
\omega_z \\
v_x \\
v_y \\
\end{bmatrix}

 =
\begin{bmatrix}
\omega_z \\
-D\omega_z + v_x \\
v_y \\
\end{bmatrix}

=
\begin{bmatrix}
\omega_z \\
r \dot{\phi_1} \\
0 \\
\end{bmatrix},
$$

$$
V_r = 
\begin{bmatrix}
1 & 0 & 0\\
D & 1 & 0\\
0 & 0 & 1\\
\end{bmatrix}

\begin{bmatrix}
\omega_z \\
v_x \\
v_y \\
\end{bmatrix}

 =
\begin{bmatrix}
\omega_z \\
D\omega_z + v_x \\
v_y \\
\end{bmatrix}

=
\begin{bmatrix}
\omega_z \\
r \dot{\phi_2} \\
0 \\
\end{bmatrix}
$$


<!-- $$
m = \frac{(S_1-E_1)+(S_2-E_2)+...+(S_{10}-E_{10})}{10}
\\
Std = \sqrt{\frac{(S_1-E_1)^2+(S_2-E_2)^2+...+(S_{10}-E_{10})^2}{10}}
$$ -->


Therefore, we can get left and right wheel speed $\dot\phi_r$ and $\dot\phi_l$ as below 

$$
\dot\phi_l  = 
\frac{1}{r}
\begin{bmatrix}
-D & 1 & 0\\
\end{bmatrix}
\begin{bmatrix}
\omega_z \\
v_x \\
v_y \\
\end{bmatrix},
$$

$$
\dot\phi_r  = 
\frac{1}{r}
\begin{bmatrix}
D & 1 & 0\\
\end{bmatrix}

\begin{bmatrix}
\omega_z \\
v_x \\
v_y \\
\end{bmatrix}
$$

- #### Wheels Speed to Twist
Using above equations, we can get matrix 

$$
\begin{bmatrix}
\dot\phi_l\\
\dot\phi_r\\
\end{bmatrix}

= H
\begin{bmatrix}
\omega_z \\
v_x \\
v_y \\
\end{bmatrix}
$$

$$ 
H = 
\frac{1}{r}

\begin{bmatrix}
-D & 1 & 0\\
D & 1 & 0\\
\end{bmatrix}
$$

Then we can use pseudo-inverse $H^+$ to calculate body twist as follow equations:

$$
H^+ =
H^T (HH^T)^{-1}
$$


$$
V_b = H^+ 
\begin{bmatrix}
\dot\phi_l\\
\dot\phi_r\\
\end{bmatrix}
$$


### fake encoders:
In rigid2d library, I create fake encoder class to creates fake joint states for differential drive robots on simulation. 

### Odometry: 
Also I create a odometry class to update robot odometry transforms based on joint states.

### waypoints;
I create waypoints class helps to calculate sequence of forward and angular velocities requires for a robot to travel between waypoints.

---

## Pentagon Path-Planing
This is a example of using rigid2d library. I controlled a turtle to move in a pentagon path continuously (As shown on Figure) by sending linear and angular velocities. I create fake joint states, update its odometer, and simulate diff drive robot on rivz.  
 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/review/img/turtle_way_combine.gif?raw=true" alt="drawing" /> </p>

*<center>Figure : Control a turtle move in a pentagon path </center>*

I predict turtle pose based on the sending velocity command (linear and angular velocities), 
The below chart is calculated error between predict turtle pose and real turtle position in x, y and theta.  The errors are smaller than absolute value of 0.01 meters.
 <p align="middle"> <img src="https://github.com/luxi-huang/Turtulebot3-Navigation/blob/review/img/turtle_way_error.png?raw=true" alt="drawing" /> </p>  

 ---









