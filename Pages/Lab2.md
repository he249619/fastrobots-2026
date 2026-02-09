--- 
title: "Lab 2"
description: "Lab Description and Report"
layout: default
---

# Lab 1: Inertial Measurement Unit

<iframe width="560" height="315" src="https://www.youtube.com/embed/DG_8jBnNdu4"
  title="ECE 4160: Lab 1A Blink" frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>

# <img src="Images/Lab 1/lab1a_task3.png" alt="Task 3, Image 1" style="width: 610px; height: 397px"/>

```cpp
Serial.printf("temp (counts): %d, vcc/3 (counts): %d, vss (counts): %d, time (ms) %d\n", temp_raw, vcc_3, vss, millis());
```

# Lab Tasks
## Set up the IMU
### Picture of your Artemis IMU connections
### Show that the IMU example code works
### AD0_VAL definition discussion
### Acceleration and gyroscope data discussion (pictures recommended)
## Accelerometer
### Image of output at {-90, 0, 90} degrees for pitch and roll (include equations)
### Accelerometer accuracy discussion
### Noise in the frequency spectrum analysis
#### Include graphs for your fourier transform
#### Discuss the results
## Gyroscope
### Include documentation for pitch, roll, and yaw with images of the results of different IMU positions
###  Demonstrate the accuracy and range of the complementary filter, and discuss any design choices
## Sample Data
### Speed of sampling discussion
### Demonstrate collected and stored time-stamped IMU data in arrays
### Demonstrate 5s of IMU data sent over Bluetooth
## Record a Stunt
### Include a video (or some videos) of you playing with the car and discuss your observations


