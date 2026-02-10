--- 
title: "Lab 2"
description: "Lab Description and Report"
layout: default
---

# Lab 2: Inertial Measurement Unit

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

### Accelerometer accuracy discussion

When the accelerometer returns measurements, the precision of the measurements depends on the full-scale range of the IMU, which is programmable. In my program, the full-scale range is such that the accelerometer reports acceleration values between ±2g. This means that a value of 1g is reported as the integer value 16,384. Therefore, when the IMU flat on the table, one would expect the z-acceleration to be +16,384 when facing up, and -16,384 when facing down. However, my IMU reported about 17,114 when facing upwards, on average, and -15,782 when facing down. Assuming that any offset from the expected output of the IMU is linear with respect to its orientation, then I can determine how far away the data is from the theoretical values in the face-down and face-up position, average them, and add this offset to all of the acceleration data. Therefore, I will subtract about 660 to all acceleration data collected moving forward. This is a correction of about 0.04g's.

### Image of output at {-90, 0, 90} degrees for pitch and roll (include equations)

In order to calculate the pitch and roll of the IMU, two very simple equations are needed. By using geometry and trigonomotry on a simple model of the IMU, it can be determined that the equation to determine the pitch of the IMU is: **atan2(x-acceleration, z-acceleration)**, and the equation for roll is **atan2(y-acceleration, z-acceleration)**. As it can be seen, these two equations are both dependent on the z-acceleration, which couples them and can lead to undesireable affects. For example, whenever either the pitch or roll are at ±90 degrees, the other measurement isn't at 0 degrees like we would expect. This is because when, for example, pitch is a +90 degrees, the x-acceleration is positive and the z-acceleration is zero. This would correspond with **atan2(infinity)**, which returns +90 degrees. However, the 0 z-acceleration also causes the roll equation to diverege, but to -90 degrees instead. 

This behavior is due to the mathematical equations of pitch and roll and is not related to sensor itself. See below for more examples of this.

# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_-90.png" style="max-width:50%"/>
# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_0.png" style="max-width:50%"/>
# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_90.png" style="max-width:50%"/>

# <img src="Images/Lab 2/a_roll_and_pitch_roll_-90.png" style="max-width:50%"/>
# <img src="Images/Lab 2/a_roll_and_pitch_roll_0.png" style="max-width:50%"/>
# <img src="Images/Lab 2/a_roll_and_pitch_roll_90.png" style="max-width:50%"/>


### Noise in the frequency spectrum analysis

**Do this discussion**

#### Include graphs for your fourier transform

**noisy_acc_when_flat, noisy_acc_when_flat_x_freq, noisy_acc_when_flat_y_freq, noisy_acc_when_flat_z_freq, noisy_acc_angles_when_flat, noisy_acc_angles_when_flat_pitch, noisy_acc_angles_when_flat_roll**

**filtered_acc_when_flat, filtered_acc_when_flat_x, filtered_acc_when_flat_y, filtered_acc_when_flat_z, filtered_acc_angles_when_flat, filtered_acc_angles_when_flat_pitch, filtered_acc_angles_when_flat_roll**

#### Discuss the results

## Gyroscope
### Include documentation for pitch, roll, and yaw with images of the results of different IMU positions
###  Demonstrate the accuracy and range of the complementary filter, and discuss any design choices

## Sample Data
### Speed of sampling discussion
### Demonstrate collected and stored time-stamped IMU data in arrays
**Look at actual arduino code for this**
### Demonstrate 5s of IMU data sent over Bluetooth

## Record a Stunt
### Include a video (or some videos) of you playing with the car and discuss your observations


