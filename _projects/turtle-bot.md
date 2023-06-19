---
layout: page
title: Delivery Robot
description: Design of the state estimation and controls of a turtle bot for a mail delivery simulation.
img: assets/img/turtlebot.PNG
importance: 1
category: Projects
---

### Technologies Used
- ROS
- Python
- PID Controller
- Bayesian Localization
- Raspberry Pi


Github Repository: <a href="https://github.com/IsmailOuazzani/Delivery-Robot">Delivery-Robot</a>

# Introduction
We design the control to simulate a Mail Delivery Robot, as the final project of our robotics class.
We are given a topological map of an office track, and enable a robot to stop at the instructed offices by implementing both Bayesian Localization and a PID controller.  The offices, numbered from 1 to 12 are unique but their color (sensor reading) is not. They are connected by a black line that the robot follows.

<div id="a" align="center">
<img src="https://user-images.githubusercontent.com/74887266/223531741-035bf93e-006e-4a24-82b2-7580a2d9e9f0.PNG"  width="30%" height="15%">
  
  <h5>Topological map</h5>
</div>

<iframe src="https://drive.google.com/file/d/18__z6zrBNwR1Fe-d58yG_F8b9K7sWxUA/preview" width="640" height="480" allow="autoplay"></iframe>

# Robot Platform
We worked with a TurtleBot3 Waffle Pi equipped with the Pi Camera, pointed downward to examine
the area underneath the robot. The camera is useful when mounted this way as it can be used for line
following and floor color detection. The different sensors (Camera and motors) communicated with
each other through a ROS infrastructure that was provided to us. We were provided with the code for perception and data extrapolation from the Pi camera.

<div id="a" align="center">
<img src="https://www.smartrobotworks.com/images/Platform/TurtleBot3/TurtleBot3_WafflePi.png"  width="30%" height="15%">
</a>

# Solution Strategy
## Line following
We use the Pi camera to detect the current color the robot is on. If the color detected is black, we expect
it to be on the line, otherwise, we assume the robot to be in an office.

To follow the line we use the camera to determine whether the line is centered in the frame. If the
line is not, we assume that the robot’s trajectory needs to be corrected and we use a PID controller
to perform the correction. If properly tuned, we assume the PID controller to always maintain the
robot centered on the line.

If the color detected by the camera is not black, we assume the robot has reached an office. As
the offices can be crossed by traveling forward in a straight manner, we decided to turn off the PID
in these areas to avoid twitching. The offices are uniformly colored and span the whole field of view
of the camera. It is therefore impossible for the robot to correct its trajectory given the image shown.
Leaving the PID controller on when the robot is in an office, would imply uncontrollable path deviation.
Turning off the PID controller in the offices, implies blindly moving forward for some time. If our
PID controller is properly tuned, the robot will be centered on the black line before entering an office,
and it will therefore find the black line again after exiting the office, as the black line doesn’t change
direction within the office span.

## Robot localization
For Robot localization we use Bayesian localization. Once we
encounter an office we use the Bayesian model to keep track
of the previous colors and predict the current office given the
current color measured. We initialize the initial position to
be uniformly probable across all the states since the robot can
start from any location. We expect the model to converge to the
right position after some offices have been visited. We therefore keep looping around the track until the model becomes
confident enough in its current state.

## Comments on implementation
Our PID controller was tested in previous labs so we expect it
to be straightforward to implement. We will first implement
our Bayesian model on its own, and ensure its functioning by
simulating a virtual track. This will allow us to debug it without having to test it on the robot. We expect color detection to
be the harder part of the lab. This is the solution we identified.
The color detected by the camera at any point is one of
the following: black, blue, green, yellow or orange. We will
therefore provide the RGB values of such colors to our robot.
We calibrate the RGB values corresponding to the colors of the
offices based on the mean of several observations taken form the
track. When we run the robot we will then compare the colour measured by the robot to the provided
ones, and determine which on it is the most similar to. We have different ideas to determine such
similarity. We will converge on the best one in later sections. We will test

- **Norm of Differences:** Take the difference between the measured value and one of the provided
values, and calculate the norm of the final vector. The measured value will be the one with the
smallest norm.
- **Cosine Similarity:** Determine the angle between the measured rgb value and the provided
values. The measured value will be the one with the smallest angle.

# Methods

## Bayesian Localization

We implemented the code by following the theory behind Bayesian localization. When testing it on
the robot, we noticed that no matter the starting position, it took the same number of offices visited
to confidently converge to the correct one. We therefore decided to implement a threshold: our robot
would ”be certain” of its localization only after exploring three offices. This simplified the logic of our
code while still reducing the risk of stopping at the wrong office.


## Improving color detection
### HSV format
We noticed that the RGB values of the colors were very close to each other, and both methods listed
above failed to determine similar colors consistently. We therefore switched to HSV values as they
allowed more distinction between colors than RGB. Out of the options described earlier, we chose
cosine similarity as it provided more accurate results.

### Robot add-ons
We tried other approaches to reduce the error in color detection such as using a physical cone made
of paper to have constant lighting, or using a portable light placed on top of the camera. Both of
those approaches made color detection slightly worse. The failure of the cone to produce accurate
measurements was likely caused by the fact that it appeared in the camera field of view and it distorted
the values of the color detected. The portable light made the floor too shiny and negatively impacted
the color detected as well.
### Color Filtering
When entering a new office, we noticed a transient period during which multiple colors were detected
one after the other. We therefore created a list (COLOR-LIST) containing the last K detected colors
measured by the camera. When a new color was detected, we appended it to the end of the list and
popped the first element. We only changed the current color variable, indicating the color the robot
thought it was on, when all the colors in that list were the same. For example, if the robot was on
the line and it is now starting to see an orange office. COLOR-LIST, if K = 5, will probably look like
[”black, ”black, ”yellow”, ”orange”, ”orange”]. In this case current color is still ”black”. Only when
COLOR-LIST = [”orange”, ”orange”,”orange”, ”orange”,”orange”] the current color will be equal to
”orange”.
Even after this modification, our robot struggled to detect the black line after specific colors. For
example, after crossing a yellow office, our robot tent to confuse the line with blue. Therefore, we
hard-coded the fact that after visiting an office, the next color measured had to be black, and we
disregarded all other measurements.
### PID Control Activation
The introduction of color filtering introduced some problems in our PID system as it was disengaging
too late: when entering the office, the robot would twitch. To fix this issue, we stopped the PID
controller when at least 27% of the values contained in COLOR-LIST were not black (we found this
value through trial and error).
Similarly, we wanted to start the PID at the right time after leaving an office so that if the robot
deviated while moving across the office, it could be corrected before it was too late. We looked at the
mean index of ”black” labels in the list. If the mean was toward the end of the list, it meant that we
were getting new black measurements, i.e. that the robot was leaving the office, so we turned on the
PID.

# Performance Analysis
Our Bayesian localization code successfully converged
after 3 offices were visited and our robot was able to detect the office color with 100% accuracy. The
following graphs show a sample robot prediction when it starts at office 9. The initial distribution
is uniform. The robot measures orange then yellow. It converges to office 10 as {9,10} is the only
sequence containing orange then yellow in the topological map (See introduction).


<img src="https://user-images.githubusercontent.com/74887266/223527480-1ce1b6f6-561b-44bc-83c3-3acbec4129c3.png" width="200">
<br>
<img src="https://user-images.githubusercontent.com/74887266/223527502-4704b784-12f7-4df5-ba56-8db89112f31f.png" width="200">
<br>
<img src="https://user-images.githubusercontent.com/74887266/223527515-79d90084-4b5d-4975-b3c5-7b148af6d7ad.png" width="200">


We only noticed some small problems with our PID controller and the color orange. On orange
officies, the robot would not turn of PID in time in 40% of the cases. We did not have time to debug
this last problem, although we hypothesize that something might be missing from our PID controller
activation logic.


# Future Improvements
In terms of hardware, a better camera (higher resolution) can improve both PID control and color
detection. Fusing the information from multiple cameras could also decrease the error.
Another improvement for color detection is the use a Decision-based tree model to assign a color
based on the sensor measurement. It would be more resilient against changes in lighting and color
shade. It would also give insight into why a color is selected instead of another one - by using metrics
such as the SHAP values - which could help to build a better test setup.
We noticed that color detection is sensitive to lighting conditions (such as a person moving above
the robot), and abrupt changes can cause errors. To fix this issue, we could use a physical cone around
the camera, combined with a portable light, and modify the algorithm that measures the color to only
measure a subset of the entire image (where we don’t see the cone).

# Conclusion
Altogether, this proof-of-concept demonstration was an opportunity for us to deepen our understanding of Bayesian localization while strengthening our skills in prototyping and debugging, all of which
are essential to becoming a seasoned roboticist.
A key idea that we learned is to build incrementally small components that fit together. We could
have tried to build the entire project from the start, but having built and tested each element separately - the PID control, color detection, and bayesian localization - allowed us to pinpoint issues and
make steady progress, which are important in terms of efficiency.