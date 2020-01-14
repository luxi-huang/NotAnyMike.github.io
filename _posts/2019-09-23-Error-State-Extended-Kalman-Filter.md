---
layout: inner
title: 'Unsupervised Depth Estimation Explained'
date: 2019-09-23 14:15:00
categories: development
type: hidden
tags: Maths Deep-learning Computer-vision
featured_image: '/img/posts/Depth/depth.jpg'
comments: true
project_link: 'https://github.com/notanymike/HRL'
button_icon: 'github'
button_text: 'Visit Project'
lead_text: 'Explaining how unsupervised depth estimation works and a re-implementation of the original paper'


---

# A quick review of Error State - Extended Kalman Filter

Lately, there have been several interesting papers [^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11][^12][^13][^14][^15] for depth estimation using only images and without any supervision. Some approaches use stereo images, some others use scale consistency to improve results, there are implementations with multiple masks as well as a very simple defined mask to filter out moving objects or occluded or new visible objects.

---

# Explanation

### Notation

We will use Tait-Bryan Euler rotations, that is $$\alpha, \beta, \gamma$$ 

## State

The state $$x_t$$ is composed by $$x$$ and $$y$$ coordinates, the yaw $$\yaw$$, velocity $$v$$ and steering angle $$st$$. Have in mind thath $$x_t$$ is the state of the filter while $$x$$ is the value in the $$x$$-axis. The tracked orientation is only composed by the yaw $$\phi$$, we are only modelling a 2D world, therefore we do not care about the roll $$\theta$$ or pitch $$\psi$$. And finally we added the $$\text{steering angle}$$ which is important to predict the movement of the car.


$$
x_t= \left[\begin{matrix}
x\\y\\\text{yaw}\\v\\\text{steering angle}
\end{matrix}\right]
$$


## Sensors

I want to exemplify a Kalman Filter using different kind of sensors. This is slightly more interesting problem, also Kalman Filter is the right tool to measure several different sensor information.

#### 1. Localisation reading

This measurement contains the global measurements that avoid the system of drifting, with out this measurement, the system is also called Dead reckoning. Dead reckoning or using a Kalman Filter without a global measurement (like $$x,y$$ coordinates in this case) is prone to cumulative errors, that means that the state will slowly diverge from the true value.

#### 2. Inertial Measurement Unit 

#### 3. Wheel sensors



To transform/rotate an object, it is necessary to convert the 6DoF into the transformation matrix, there are several interesting materials out there regarding how to do it. The result of that operation is the following matrix



$$
[R|\boldsymbol t] = \begin{bmatrix}R&\boldsymbol t\\\boldsymbol 0 &1\end{bmatrix}
$$



Where $$R$$ is the rotation matrix (have in mind that $$R$$ is not the same as the Euler angles but comes from them) and $$\boldsymbol t$$ is the translation in the 3D world $$[t_x,t_y,t_z]^T$$.

Finally, $$z_c$$ is the depth from the camera frame and can be considered the scaling factor. The subscript $$c$$ means it is in respect to the camera frame, where for instance $$w$$ would mean it is in the world coordinates.

Usually, the models cited, use a target and a source image. Some use two sources to improve the results, nonetheless, the concept is the same. The target is the image being predicted and the source is the image used to predict the target. The main equation of *Zhou et al.* [^1] follows the following logic. The first step is to covert the **coordinates** of the target image ($$x$$-$$y$$ pixels, a normal mesh grid) into world coordinates, by doing this



$$
K^{-1}I_{grid} \cdot {z_{\text{w}}^\text{target}} \leftarrow \text{This is in world coordinates}
$$



$$I_\text{grid}$$ are all the coordinates of the pixels. This operation is exactly the opposite showed above, instead of projecting 3D world points into an image, this converts 2D images into a cloud point. Because going from 3D to 2D we drop information (mainly the depth), going from 2D to 3D need extra information to reconstruct the 3D point cloud, we add this lost information (depth) when multiplying by $$z_w^\text{target}$$. If you solve for the 3D world coordinates in the first equation, this is the same operation with no translation nor rotation.

Next, this result is translated again, from the target location to the source location using the camera movement $$[K\mid\boldsymbol t]_{\text{target}\to\text{source}}$$. Then it is converted again into pixel coordinates by multiplying the result by the intrinsic matrix $$K$$.



$$
I_\text{source_coords} \leftarrow \underbrace{K[R|\boldsymbol t]_{\text{target}\to\text{source}}}_{\texttt{proj_tgt_cam_to_src_pixel}}\underbrace{\left( K^{-1}I_{grid}* {z_{\text{camera}}^\text{target}} \right)}_{\texttt{cam_coords}}
$$



With this, we have "matched" the coordinates from the target image with coordinates from the source image. In other words, we know where each pixel in the target image should go in the source image. That means that we can reconstruct the target image from pixels from the source image. The common way of doing this is using bilinear interpolation, which could be considered as a "convex" combination of the pixels' colours where the weights are the distance between the pixels coordinates.



$$
I_\text{projected_target} = B(I_\text{source_coords},I_\text{source})
$$



This is called inverse warp because instead of sending the pixels from source to target, it matches the pixels from target to source and from there reconstruct the target. If I am not mistaken (if I am, correct me) in this way the projection into target pixels does not have fractional pixels (e.g. $$(3.3,2.5)$$) and the bilinear interpolation is done in the source image.

This reconstruction will have several dead pixels. Not even in a perfect world, this matching operation could be a bijection between the two frames. For example pixels of the scene that were not visible in the source but were visible in the target will not have a match in the other image. Apart from these dead pixels, objects moving (dynamic objects) will have a different position in the next frame. For them, estimating depth from the target is not enough, they will also require estimating their motion. All those impossible to match pixels are also predicted and the list of pixels that are not predictable are put in a Mask $$M$$. If $$M$$ is all the non-predictable pixels (due to occlusion, dynamic objects, etc.) then $$1-M$$ is all the predictable pixels. There are different ways of predicting these un-matchable pixels, for an interesting approach see *Bian et al.* [^13]

From there, basically the loss is the reconstruction loss between the predictable pixels between the projected image and the target image. That is:



$$
\mathcal L = \sum_\text{pixels}|I_\text{projected_target}-I_\text{target}|(1-M)
$$



For this, it was necessary to estimate the depth of the target $$z_\text{camera}^\text{target}$$ and the movement of the camera (ego-motion) $$[R\mid\boldsymbol t]$$. That is the basic approach, there we were given the intrinsic matrix, one extra step is to estimate the intrinsic matrix $$K$$ as well, which several approaches do. It is also possible to differentiate between unpredictable pixels and dynamic objects and try to predict its movement.

Finally, it is important to mention that the entire loss function includes terms for smoothness as well as regularisation terms, which I don't cover here because the focus is depth estimation.

(if you see I have made a mistake, don't hesitate to tell me).



---

[^1]: Zhou, T., Brown, M., Snavely, N., & Lowe, D. G. (2017). Unsupervised Learning of Depth and Ego-Motion from Video. *2017 IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*, 6612–6619. https://doi.org/10.1109/CVPR.2017.700
