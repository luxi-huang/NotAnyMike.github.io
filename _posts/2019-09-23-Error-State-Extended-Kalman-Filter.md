---
layout: inner
title: 'Unsupervised Depth Estimation Explained'
date: 2019-09-23 14:15:00
categories: development
type: hidden
tags: Maths Sensor-Fusion Kalman-Filter
featured_image: '/img/posts/Depth/depth.jpg'
comments: true
project_link: 'https://github.com/notanymike/HRL'
button_icon: 'github'
button_text: 'Visit Project'
lead_text: 'Explaining how unsupervised depth estimation works and a re-implementation of the original paper'


---

# A "quick" review of Error State - Extended Kalman Filter

There is an incrediable lack of resources explaining with detail how Kaman Filter (KF) works. Imagine now the lack of resources explaining a more complex KF as the Error-state Extended Kaman Filter (ES-EKF). On the other side, there are fair ammount of blogs covering the Unscented Kalman Filter. In this post I will focus on the ES-EKF and leave UKF alone for now. One of the only blogs regarding a linear KF worth reading is [kalman filter with images](https://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/) which I totally recommended. Here I will cover with more details the whole linear kalman filter equations and how to derive them. After that I will explain how to transform it into an Extended KF (EKF) and then how to transform it into a Error-state Extended KF (ES-EKF).

### Notation

We will use Proper Euler angles to note rotations, that will be is $$\alpha, \beta, \gamma$$, we are only interested on 2D rotations therefore we will use the z-x'-z'' representation in which $$\alpha$$ represents the yaw (the representation does not matter as far as the first rotation happens in the $$z$$ axis). The steering angle will be noted by $$\delta$$.

# Explanaition

The Kalman Filter is used to keep track of certain variables and fuse information coming from other sensors such as Inertial Measurement Unit (IMU) or Wheels or litterally any other sensor. It is very common in robotics because it fuses the information according how certain the measurments are. Therefore we can have several sources of information, some more reliable than others and a KF takes that into account in order to keep track of the variables we are interested.

## State

The state $$x_t$$ we are interested on tracking is composed by $$x$$ and $$y$$ coordinates, the heading of the vehicle or the yaw $$\alpha$$, the current velocity $$v$$ and steering angle $$\delta$$. Have in mind that $$x_t$$ is the state of the filter while $$x$$ is the value in the $$x$$-axis. The tracked orientation is only composed by the yaw $$\alpha$$, we are only modelling a 2D world, therefore we do not care about the roll $$\beta$$ or pitch $$\gamma$$. And finally we added the steering angle $$\delta$$ which is important to predict the movement of the car. Therefore the state in timestep $$t$$ is


$$
s_t= \left[\begin{matrix}
x\\y\\\theta\\v\\\delta
\end{matrix}\right]
$$

KF can be understood in two steps, update and predict step. In the predict step, using the tracked information we predict where will the object move in the next step. In the update step we update the belief we have about the variables using the external measurements comming from the sensors.

## Sensor

I want to exemplify a Kalman Filter using different kind of sensors, but that will be repeating equations, so I think is more interesting using a Kinmatic model instead of more sensors (if I have time I will expand this post). Nonetheless keep in mind that a KF can handle any number of sensors, so far we are going to use the localization measurement comming from a gps + pseduo-gyro.

This measurement contains the global measurements ($$x,y$$) that avoid the system of drifting. This system (with out global variables) is also called Dead reckoning. Dead reckoning or using a Kalman Filter without a global measurement (like $$x,y$$ coordinates in this case) is prone to cumulative errors, that means that the state will slowly diverge from the true value.

## Prediction Step

We will track the state as a multivariable gaussian distribution with mean $$\mu$$ and covariance $$P$$. Where $$\mu_t$$ will be the expected value of our state. $$\mu_t$$  will be the expected value of the state using the information available (i.e. the mean of $$s_t$$). And state will have a covariance matrix $$P$$  which means how certain we are about our prediction. We will use $$\mu_{t-1}$$ and $$u$$ to predict $$ \mu_t$$. Here $$u$$ is a control column-vector of any extra information we can use, for example steering angle if we can have access to the streering of the car or the acceleration if we have access to it. $$u$$ can be a vector of any size.

We will try to model everything using matrices but for now we will use scalars, the new value of the state in $$t$$ will be


$$
\begin{align}
x_t &= x_{t-1} + v\Delta t \cos \theta\\
y_t &= y_{t-1} + v\Delta t \sin \theta\\
\theta &= \theta_{t-1}\\
v_t &= v_{t-1}\\
\delta_t &= \delta_{t-1}
\end{align}
$$


Here we are making a simplifying assumptions about the world. First the velocity $$v$$ and the steering $$\delta$$  of the next step will be the same as before which is a weak assumption. The strong assumption is that the heading or yaw of the car $$\alpha$$ is also the same. We can incorporate the kinematic model here to make the prediction more robust. But that will be adding non-linearities (and so far it is a linear KF) so for now let's work with a simple environment and later on we can make things more interesting.

This prediction can be re-formulated in matrix for as


$$
\mu_t = F\mu_{t-1} + Bu
$$


Where $$u$$ only zeors and $$B$$ is a linear transformation from $$u$$ into the same form of the state $$s$$. Also $$F$$ would be (F has to be linear so far, in the EKF we will expand that to include non-linearities)


$$
F = \left[\begin{matrix}
1 & 0 & 0 & \Delta t\cos\theta & 0 \\
0 & 1 & 0 & \Delta t\sin\theta & 0\\
0 & 0 & 1 & 0 & 0\\
0 & 0 & 0 & 1 & 0\\
0 & 0 & 0 & 0 & 1
\end{matrix}\right]
$$


This will result in the same equations but using matrix notation. Rember now that we are modeling $$s$$ as a multivariable gaussian distribution and we are keeping track of the mean of the state $$s$$ and the covariance of the state $$P$$. We already updated the mean of the state, now we have to update the covariance of the state. Every time we predict we make small errors which add noise and results in a slightly less accurate prediction. The covariance $$P$$ has to reflect this reduction in certainty. The way it is done with gaussian distributions is that the distribution gets slightly more flat (i.e. the covariance "increase"). 

In a single-variable gaussian distribution $$y \sim \mathcal N (\mu',\sigma^2) $$ the variance has the property that $$\text{var}(ky) = k^2\text{var}(y)$$, where $$k$$ is a scalar. In matrix notation that is $$P_t = FP_{t-1}F^T$$, but we have to take into account that we are adding $$Bu$$, where $$u$$ is the control vector but it is also a gaussian variable with covariance $$Q$$. Good thing about gaussians is that the covariances of a sum of gaussians is the sum of the covariances. Having this into account we have.


$$
P_t = FP_{t-1}F^T+BQB^T
$$


And with this we have finished prediction the state and updating its covariance.

## Update step

In the update step we receive a measurement $$z$$ coming from a sensor. We use the sensor information to correct/update the belief we have about the state. The measurement is a random variable with covariance $$R$$. This is where things get interesting. In this case we have two gaussians variables, the state best estimate $$ \mu_t$$ and the measurement reading $$ z$$. 

The best way to combine two gaussians is by multiplying them together. By multiplying them together, if certain values have hight certainty in both gaussians, the result will be also a high in the product (very certain). If both values have low certainty, the product will be even lower. And if If only one is high and the other is not, then the result will lay between high and low certainty. So multiplication of gaussians merge the information of both gaussians taking into account how certain the values are (covariance).

The equations derived from multiplying two multivariate gaussians are similar to the single variable case. We will derive them here and generalize that to matrix form.

Let's suppose we have $$x_1 \sim \mathcal N (\mu_1,\sigma_1^2)$$ and $$x_2\sim\mathcal N(\mu_2,\sigma_2^2)$$. Have in mind that both $$x_1$$ and $$x_2$$ live in the same vector space $$x$$, therefore 


$$
\begin{align}
	p(x_1) = \frac 1 {\sqrt{2\pi\sigma_1^2}}e^{-\frac{(x-\mu_1)^2}{2\sigma_2^2}} & & p(x_2) = \frac 1 {\sqrt{2\pi\sigma_2^2}}e^{-\frac{(x-\mu_2)^2}{2\sigma_2^2}}
\end{align}
$$


by multiplying them together we obtain


$$
\frac 1 {\sqrt{2\pi\sigma_1^2}}e^{-\frac{(x-\mu_1)^2}{2\sigma_1^2}}\frac 1 {\sqrt{2\pi\sigma_2^2}}e^{-\frac{(x-\mu_2)^2}{2\sigma_2^2}}
$$


We also now about a very useful property of gaussians, that is the product of gausians is also a gausian. Therefore, to know the result of fusing both gausians together we have to write the equation above in a gausian form.


$$
\begin{align}
&=\frac 1 {\sqrt{2\pi\sigma_1^2}}e^{-\frac{(x-\mu_1)^2}{2\sigma_1^2}}\frac 1 {\sqrt{2\pi\sigma_2^2}}e^{-\frac{(x-\mu_2)^2}{2\sigma_2^2}}\\

&=\frac 1 {2\pi\sigma_1^2\sigma_2^2}e^{-\left(\frac{(x-\mu_1)^2}{2\sigma_1^2}+\frac{(x-\mu_2)^2}{2\sigma_2^2}\right)}\\
\end{align}
$$


Because we know the result will be a gausian distribution, we do not care about constant values (e.g. $$2\pi\sigma_1^2$$), in fact we only care about the exponent value, which I have to transform it into something similar to 


$$
\frac{(x-\text{something})^2}{2\text{something else}^2}
$$


 

Where $$\text{something}$$ will be the new mean and $$\text{something else}^2$$ will be the new covariance after multiplication. Therefore we will ignore all the other terms and focus on the exponent value.


$$
\begin{align}
\frac{(x-\mu_1)^2}{2\sigma_1^2}+\frac{(x-\mu_2)^2}{2\sigma_2^2} &= \frac{\sigma_2^2(x-\mu_1)^2+\sigma_1^2(x-\mu_2)^2}{2\sigma_1^2\sigma_2^2}\\
&= \frac{\sigma_2^2x^2-2\sigma_2^2\mu_1x+\sigma_2^2\mu_1^2   +   \sigma_1^2x^2-2\sigma_1^2\mu_2x+\sigma_1^2\mu_2^2}{2\sigma_1^2\sigma_2^2}\\
&= \frac{x^2(\sigma_2^2+\sigma_1^2)-2x(\sigma_2^2\mu_1+\sigma_1^2\mu_2)}{2\sigma_1^2\sigma_2^2}+\frac{\sigma_2^2\mu_1^2+\sigma_1^2\mu_2^2}{2\sigma_1^2\sigma_2^2}\\
&=  \frac{(\sigma_2^2+\sigma_1^2)}{2\sigma_1^2\sigma_2^2}\left(x^2-2x\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)+\frac{\sigma_2^2\mu_1^2+\sigma_1^2\mu_2^2}{2\sigma_1^2\sigma_2^2}\\
\end{align}
$$


The term on the right can be ignore because it is constant and goes out of the exponent. And the term in parenthesis resembles to a perfect square trinomial lacking the last squared term.


$$
\begin{align}
&=  \frac{(\sigma_2^2+\sigma_1^2)}{2\sigma_1^2\sigma_2^2}\left(x^2-2x\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)\\
&=  \frac{(\sigma_2^2+\sigma_1^2)}{2\sigma_1^2\sigma_2^2}\left(x^2-2x\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2} + \left(\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)^2 - \left(\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)^2 \right)\\
&= \frac{(\sigma_2^2+\sigma_1^2)}{2\sigma_1^2\sigma_2^2}\left(\left(x-\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)^2 - \left(\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma^2+\sigma_2^2}\right)^2 \right)\\
\end{align}
$$


Ignoring the second term because it is also a constant, the final result of the exponent value is


$$
\frac{(\sigma_2^2+\sigma_1^2)}{2\sigma_1^2\sigma_2^2}\left(x-\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma_1^2+\sigma_2^2}\right)^2 = \frac{\left(x-\frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma_1^2+\sigma_2^2}\right)^2}{\frac{2\sigma_1^2\sigma_2^2}{(\sigma_2^2+\sigma_1^2)}}
$$


In fact this final form does resemble a Gaussian distribution. The new mean will be what is in the parenhesis with $$x$$ and the new covariance will be the denominator divided by 2. To simplify thing further along the way, we will re write things so
$$
\begin{align}
\mu_{\text{new}} &= \frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma_1^2+\sigma_2^2}\\
&= \mu_1 + \frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2}{\sigma_1^2+\sigma_2^2} - \mu_1\\
&= \mu_1 + \frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2-\mu_1(\sigma_1^2+\sigma_2^2)}{\sigma_1^2+\sigma_2^2}\\
&= \mu_1 + \frac{\sigma_2^2\mu_1+\sigma_1^2\mu_2-\mu_1\sigma_1^2-\sigma_2^2\mu_1}{\sigma_1^2+\sigma_2^2}\\
&= \mu_1 + \frac{\sigma_1^2(\mu_2-\mu_1)}{\sigma_1^2+\sigma_2^2}\\
&= \mu_1 + K(\mu_2-\mu_1)
\end{align}
$$


where $$K = \sigma_1^2/(\sigma_1^2+\sigma_2^2)$$. For the variance we have


$$
\begin{align}
\sigma_\text{new}&=\frac{\sigma_1^2\sigma_2^2}{(\sigma_2^2+\sigma_1^2)}\\
&=\sigma_1^2 + \frac{\sigma_1^2\sigma_2^2}{(\sigma_2^2+\sigma_1^2)} - \sigma_1^2\\
&=\sigma_1^2 + \frac{\sigma_1^2\sigma_2^2-\sigma_1^2(\sigma_2^2+\sigma_1^2)}{\sigma_2^2+\sigma_1^2}\\
&=\sigma_1^2 + \frac{\sigma_1^2\sigma_2^2-\sigma_1^2\sigma_2^2+\sigma_1^4}{\sigma_2^2+\sigma_1^2}\\
&=\sigma_1^2 + \frac{\sigma_1^4}{\sigma_2^2+\sigma_1^2}\\
&= \sigma_1^2 + K\sigma_1^2
\end{align}
$$


Now we need to transform that to matrix notation and change for the correct variables. $$\mu$$ and $$z$$ are not in the same vector space, therefore to transform $$x$$ into the same vector space as the measurement space we use the matrix $$H$$. The final result will be


$$
\begin{align}
K &= HP_{t-1}H^T(HP_{t-1}H^T+R)^{-1}\\
H\mu_t &= H\mu_{t-1}+K(z-H\mu_{t-1})\\
HP_tH^T &= HP_{t-1}H^T+KHP_{t-1}H^T
\end{align}
$$


If we take one $$H$$ out from the left of $$K$$ and we end up with


$$
\begin{align}
K &= P_{t-1}H^T(HP_{t-1}H^T+R)^{-1}\\
H\mu_t &= H\mu_{t-1}+HK(z-H\mu_{t-1})\\
HP_tH^T &= HP_{t-1}H^T+HKHP_{t-1}H^T
\end{align}
$$


We can pre-multiply the second and third equation by $$H^{-T}$$ and also post-multiply the third equation by $$H^{-1}$$, The final result turns out to be in the state vector space $$\mu$$ and not in the measurement vector space $$H\mu$$. The final result for the update step (which corresponds to the combinantion of two sources of information with different certainty levels)is


$$
\begin{align}
K &= P_{t-1}H^T(HP_{t-1}H^T+R)^{-1}\\
\mu_t &= \mu_{t-1}+K(z-H\mu_{t-1})\\
P_t &= P_{t-1}+KHP_{t-1} = (I+KH)P_{t-1}
\end{align}
$$


And that is it! The all the equations for a Linear Kalman Filter.

---

### Prediction step

$$
\begin{align}
\mu_t &= F\mu_{t-1} + Bu\\
P_t &= FP_{t-1}F^T+BQB^T
\end{align}
$$



### Update step:

$$
\begin{align}
K &= PH^T(HPH^T+R)^{-1}\\
\mu_t &= \mu_{t-1}+K(z-H\mu_{t-1})\\
P_t &= P_{t-1}+KHP_{t-1} = (I+KH)P_{t-1}
\end{align}
$$

---

## Extended Kalman Filter

In reality, world does not behave linearly. The way KF deals with non-linearities is by using the jacobian to linearize the equation. We can expand this model to a non-linear proper KF modifying the prediction step by adding a simple kinematic model, for example a bicycle kinematic model.

If we model everything from the center of gravity of the vehicle, the equations for the bycicle kinmatic model are


$$
\begin{align}
\dot x &= v\cos (\theta+\beta)\\
\dot y &= v\sin(\theta+\beta)\\
\dot \theta &= \frac{v\cos(\beta)\tan(\delta)}{L}\\
\beta &= \tan^{-1}\left(\frac{l_r\tan\delta}{L}\right)
\end{align}
$$


Where $$\theta$$ is the heading of the vehicle (yaw), $$\beta$$ is the slip angle of the center of gravity, $$L$$ is the lenght of the vehicle, $$l_r$$ is the length of the rear most part to the center of gravity and $$\delta$$ is the steering angle. In discreate time form we will have


$$
\begin{align}
x_t &= x_{t-1}+\Delta t \cdot v\cos (\theta+\beta)\\
y_t &= y_{t-1}+\Delta t \cdot v\sin(\theta+\beta)\\
\theta_t &= \theta_{t-1} +\Delta t\cdot \frac{v\cos(\beta)\tan(\delta)}{L}\\
\beta_t &= \tan^{-1}\left(\frac{l_r\tan\delta_{t-1}}{L}\right)\\
v_t &= v_{t-1}\\
\delta_t &= \delta_{t-1}
\end{align}
$$


If you define that system of equations as $$\mathbf f(x,y,\theta,v,\delta)\in\mathbb R^6$$ then we can model the whole system using $$\mathbf f$$ and $$F=\partial f_j/\partial x_i $$. We can also use the same trick with the transformation from state space $$s$$ into measurement vector space $$z$$. Before we used the matrix $$H$$ now we can use the function $$\mathbf h(\cdot)$$ and define $$H$$ as $$H=\partial h_i/\partial x_i$$. The final Extended Kalman Filter will look like

### Prediction step

$$
\begin{align}
\mu_t &= \mathbf f(\mu_{t-1}) + Bu\\
P_t &= FP_{t-1}F^T+BQB^T\\
\end{align}
$$



### Update step:

$$
\begin{align}K &= P_{t-1}H^T(HP_{t-1}H^T+R)^{-1}\\
\mu_t &= \mu_{t-1}+K(z-h(\mu_{t-1}))
\\P_t &= (I+KH)P_{t-1}
\end{align}
$$

---

## 

# Error state - Extended Kalman Filter

EKF is not a perfect method to estimate and predict the state, it will always make mistakes when predicting. The longer the ammount of sequantial predictions without updates, the bigger the acumulated error. One interesting common property of the errors is that they have less complex behaviour than the state itself. This can be seen easier in the image below. While the behaviour of the position is highly non-linear, the error (estimation - ground truth) behaves much closely to a linear behaviour.

![error](/img/posts/KF/error.png)

<span style="font-size:12px">left image taken from "Unsupervised Scale-consistent Depth and Ego-motion Learning from Monocular Video".</span>

Therefore modeling the error of the state (i.e. error-state) is more likely that will be model correctly by a linar model. Therefore, we can avoid some noise coming from trying to model highly non-linear behaviour by modeling the error-state. Let's define the error-state as $$e=\mu_t-\mu_{t-1}$$. We can approximate $$f(\mu_{t-1})$$ using the taylor series expansion only using the first derivative. Therefore $$\mathbf f(\mu_{t-1}) \approx \mu_{t-1} + Fe_{t-1}$$. Replacing this and re arranging equation we end up with the final equations for the Error state - Extended Kalman Filter (ES-EKF) is

### Prediction step

$$
\begin{align}
s_t &= \mathbf f(s_{t-1},u)\\
P_t &= FP_{t-1}F^T+BQB^T\\
\end{align}
$$



### Update step:

$$
\begin{align}K &= PH^T(HPH^T+R)^{-1}\\
e_t &= K(z-h(\mu_{t-1}))\\
s_t &= s_{t-1} + e_t\\
P_t &= (I+KH)P_{t-1}
\end{align}
$$

---



Keep in mind that now we are tracking the error state and the covariance of the error, therefore we need to predict the state $$s_t$$ and correct it by using the error-state during the update step, other wise we can estimate the state directly using $$\mathbf f$$.

(if you see I have made a mistake, don't hesitate to tell me).