---
layout: page
title: Balancing Robot
description: Implementing the controls of a 3-wheeled robot capable of balancing on a basketball.
img: assets/img/balancing.png
importance: 2
category: Projects
---

### Technologies Used
- Arduino
- C++
- P Control
- IMUs

# Introduction

We built a robot capable of balancing on a basktetball despite small disturbances. This robot resting on the ball consists of three motors positioned in a polyhedron manner in which the actuators control the maneuvering of the robot with respect to the ball. While the ball is in motion, to ensure the mechanism remains balanced on top, the actuators will be activated to counteract the ball’s movement. An Arduino Mega is used to calculate the necessary control responses bawsed on the feedback from the main sensor, namely the Inertial Measuring Unit (IMU). The actuators will be instructed to move accordingly by the Arduino board.

## Personal Contributions	
- Literature review to find equations of motion of the system
- Implementation of the control system
- Tuning of the P controller

# Video

<div id="a" align="center">
<iframe src="https://drive.google.com/file/d/1-XLWjoTJ7TukqudAsLmh2ffgU7wl32YO/preview" width="640" height="480" allow="autoplay"></iframe>
</div>

# Project Report

<object data="/assets/pdf/balancing_robot.pdf" type="application/pdf" width="100%" height="500px">
      <p>Unable to display PDF file. <a href="/assets/pdf/balancing_robot.pdf">Download</a> instead.</p>
    </object>

