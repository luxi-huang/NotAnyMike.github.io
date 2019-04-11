---
layout: inner
position: left
title: 'Solving CarRacing-v0 with PPO'
date: 2016-02-20 21:15:00
categories: RL
tags: RL OpenAI CarRacing-v0 PPO
featured_image: 'img/posts/SolvingCarRacing/1.gif'
project_link: 'https://github.com/jamigibbs'
button_icon: 'flask fas'
button_text: 'Visit Project'
lead_text: "An easy, robust and fast solution for the CarRacing-v0 environment"
---


# Solving Car Racing with Proximal Policy Optimisation
<!-- featured_image: 'img/posts/04_phantom-jekyll-1130x864-2x.png' -->

I write this because I notice a signficiant lack of information regading CarRacing environement. I aslo have expanded the environment to welcome more complex scenarios ([see more](/gym)). My intention is to publish all the information regarding how to train the model, upload the weights of my model and general tips of what to do to solve it. That will be uploaded soon ...

Meanwhile these are some of the interesting behaviours I found in my trained model


<br />
## Recovery behaviours

These are some clips of interesting recovery behaviours of the agent.

<br />

### Going backwards:

Here the agent, after recovering from slipping, returns to the track but starts going in the wrong direction.

![backwards](/site/img/posts/SolvingCarRacing/1.gif)

---

### Double recovery:

The interesting part about this one is that the agent has to go through tow recoveries, given that the first strategy didn’t work and the agent is still outside the track.

![backwards](/site/img/posts/SolvingCarRacing/2.gif)

---

### Breaking after recovering:

In this clip the agent avoids going out again by breaking and this time taking the curve slowly.

![backwards](/site/img/posts/SolvingCarRacing/3.gif)

---

### Double slipping:

I don’t know how to call this but it is not an easy recovery and an interesting one.

![backwards](/site/img/posts/SolvingCarRacing/4.gif)

---

## Some others behaviours

### Safe behaviour
This is clearly not the finished trained model, but nonetheless is an interesting behaviour where the agent does not go out of the track never, it is efficient if the agent only wants to cover all the track and don’t care about time or speed.

![backwards](/site/img/posts/SolvingCarRacing/5.gif)

---

### Breaking

You can notice how the car decelerate approaching the curve in order to take the curve right. One of the common mistakes of the very few agents in this environment is that once the agent accelerates, it does not reduce the speed and therefore ends up outside the track.

![backwards](/site/img/posts/SolvingCarRacing/6.gif)

---

### Video

Finally a video of two tracks being solved

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ev0wpVB7OEs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
