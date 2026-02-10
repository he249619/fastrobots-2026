--- 
title: "Lab 2"
description: "Lab Description and Report"
layout: default
---

# Lab 2: Inertial Measurement Unit

## Set up the IMU

### IMU Connections

The IMU connects to the Artemis Nano through a quick connect wire.

# <img src="Images/Lab 2/IMUconnection.png" style="max-width:75%"/>

### IMU Example Code

In the ICM 20948 example code, the IMU's raw values are scaled so that it has proper units with which calculations can be done on. This scaling is necessary because the IMU outputs integer values representing the acceleration, angular velocity, and magnetic field magnitude for all of x, y, and z axes, and without knowing how these seemingly random raw integer values scale to values with units the data is useless.

# <img src="Images/Lab 2/IMUdemo.png" style="max-width:90%"/>

### AD0_VAL Discussion

The value of `AD0_VAL` is what determines the final address of the IMU. By setting this value either high or low, you can change the address of this device. This is important to know about because this IMU is an I2C device, and every device on the same I2C bus must have a unique address associated with it. This means that you can have two different ICM 20948's on the same I2C bus because each one can be configured to have different addresses if the `AD0_VAL` is different for each device. By default, the `AD0_VAL` should be 1.

### Accelerometer and Gyroscope Data Discussion

While running the example code, I noticed that the accelerometer and gyroscope data behaves differently.

The accelerometer data takes on large values, comfortably ranging from 1000 mg to -1000 mg when not moving and facing upwards or downwards, respectively. When the device stays still, the accelerometer data seems to be a little bit noisy, but the noise is concentrated about some mean value. 

On the other hand, the gyroscope only reads large values when the device is rotating about one of its axes. Otherwise, the values are practically zero. This makes sense because the gyroscope measures the change in degree per second, so if the device isn't rotating then it wouldn't have an angular velocity. 

Here is an image when the IMU is facing upwards.
# <img src="Images/Lab 2/acc_gyro_data1.png" style="max-width:85%"/>

Here is an image when the IMU is facing downwards. Note the change of sign on the z-acceleration component.
# <img src="Images/Lab 2/acc_gyro_data2.png" style="max-width:85%"/>

## Accelerometer

### Accelerometer Accuracy

When the accelerometer returns measurements, the precision of the measurements depends on the full-scale range of the IMU, which is programmable. In my program, the full-scale range is such that the accelerometer reports acceleration values between ±2g. This means that a value of 1g is reported as the integer value 16,384. Therefore, when the IMU flat on the table, one would expect the z-acceleration to be +16,384 when facing up, and -16,384 when facing down. However, my IMU reported about 17,114 when facing upwards, on average, and -15,782 when facing down. Assuming that any offset from the expected output of the IMU is linear with respect to its orientation, then I can determine how far away the data is from the theoretical values in the face-down and face-up position, average them, and add this offset to all of the acceleration data. Therefore, I will subtract about 660 to all acceleration data collected moving forward. This is a correction of about 0.04g's.

### Roll and Pitch as Determined by the Accelerometer

In order to calculate the pitch and roll of the IMU, two very simple equations are needed. By using geometry and trigonomotry on a simple model of the IMU, it can be determined that the equation to determine the pitch of the IMU is: **atan2(x-acceleration, z-acceleration)**, and the equation for roll is **atan2(y-acceleration, z-acceleration)**. As it can be seen, these two equations are both dependent on the z-acceleration, which couples them and can lead to undesireable affects. For example, whenever either the pitch or roll values are at ±90 degrees, the other angle isn't at 0 degrees like we would expect. This is because when, for example, the pitch is at +90 degrees, the x-acceleration is positive and the z-acceleration is zero. This would correspond with **atan2(infinity)**, which returns +90 degrees for the pitch and is expected. However, the 0 z-acceleration also causes the roll equation to diverege, but to -90 degrees instead. 

These singularities are due to the mathematical equations of the pitch and roll and is not related to the sensor itself. See below for more examples of this.

##### Value of Pitch and Roll when Pitch is positioned at -90 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_-90.png" style="max-width:75%"/>

##### Value of Pitch and Roll when Pitch is positioned at 0 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_0.png" style="max-width:75%"/>

##### Value of Pitch and Roll when Pitch is positioned at 90 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_pitch_at_90.png" style="max-width:75%"/>

##### Value of Pitch and Roll when Roll is positioned at -90 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_roll_-90.png" style="max-width:75%"/>

##### Value of Pitch and Roll when Roll is positioned at 0 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_roll_0.png" style="max-width:75%"/>

##### Value of Pitch and Roll when Roll is positioned at 90 Degrees
# <img src="Images/Lab 2/a_roll_and_pitch_roll_90.png" style="max-width:75%"/>

### Frequency Analysis

It is very noticable in the previous graphs that the pitch and roll data are extremely noisy, which is due to the noisy accelerometer data. The image below shows the pitch and roll values when the IMU is flat and motionless on a table.

# <img src="Images/Lab 2/noisy_acc_angles_when_flat.png" style="max-width:75%"/>

This data can be analyze to determine its frequency components by computing a fourier transform of it. The following data are the frequency components of the pitch and roll.

# <img src="Images/Lab 2/noisy_acc_angles_when_flat_pitch.png" style="max-width:75%"/>

# <img src="Images/Lab 2/noisy_acc_angles_when_flat_roll.png" style="max-width:75%"/>

As it can be seen from the graphs, the measurements have non-zero frequency components for a large range of frequencies. This is undesireable and leads to the type of noisy data we are seeing. Therefore, I built a low-pass filter with a cutoff of about 2.5 Hz for the accelerometer data. By using such a low cut off frequency, the output of this filter is much less noisy, as can be seen below.

# <img src="Images/Lab 2/filtered_acc_angles_when_flat.png" style="max-width:75%"/>

# <img src="Images/Lab 2/filtered_acc_angles_when_flat_pitch.png" style="max-width:75%"/>

# <img src="Images/Lab 2/filtered_acc_angles_when_flat_roll.png" style="max-width:75%"/>

This slightly stabilizes the values of the pitch and roll, though they are still noisy. A complementary filter might be able to reduce this noise even more, and to implement one I will need data from the gyroscope within the IMU as well.

## Gyroscope

### Roll and Pitch as Determined by the Gyroscope

In order to get information about pitch and roll from the gyroscope, it is necessary to know how it reports its data. The gyroscope returns the angular velocity, in degrees per second, of one of the three axes of the IMU. In order to turn this data into something usuable, you must multiply the angular velocity of the axis of interest by the amount of time that has passed since the sensor's last sample. This would then return a change of degree in the said axis' plain, and if you continuously sum these changes over time you will get a very stable approximation for the pitch, roll, and yaw of the IMU. 

However, even though the gyroscope readings have very little noise, they will drift away from the true values that they're trying to represent. This is because in order to get the pitch, roll, or yaw, you have to continuously add upon the previous measurements, which sums the errors of each measurement. An example of this drift can be seen below, where though the IMU was motionless on a table, some of the angle measurements diverge from zero.

# <img src="Images/Lab 2/gRollPitchYawAtPitch0.png" style="max-width:75%"/>

Even with this drift, the stability of the gyroscopes data is very desireable. Take the following data, for example. When the IMU is moving, the gyroscope data is not noisy, resulting in a very clear calculation of the pitch, roll, and yaw of the IMU.

# <img src="Images/Lab 2/gRollPitchYawAtPitch+90.png" style="max-width:75%"/>

So far, we have the raw acceleration data, the low-passed acceleration data, and the gyroscope data to calculate the pitch and roll.

# <img src="Images/Lab 2/allThreeMethodsWithChangingPitch.png" style="max-width:75%"/>

But both the accelerometer and the gyroscope have benefits and costs to their usage, so we need to implement a way to combine their data and get the best of both options.

### Complementary Filter

A complementary filter must be built in order to fuse the accuracy of the accelerometer with the stability of the gyroscope. What the complementary filter will do is sum a scaled value of the accelerometer's pitch and roll, as determined by the low-pass filter, with the pitch and roll from the gyroscope data. The question then becomes how to weight the two sources properly.

The desireble trait of the accelerometer is that it is very accurate, but it is also very noisy. Additionally, due to the math behind the pitch and roll calculations, when either of those angles are at ±90 degrees, the other angle will behave as if at a singularity. Therefore, we would want the complementary filter to reduce the noise and remove the singularity from the data, but keep the accuracy. From the gyroscope, the complementary should remove the drift but keep the stability.

Therefore, the complementary filter should weigh the gyroscope much higher than the accelerometer. This is because by weighing the gyroscope data higher, the complementary filter will prioritize stability and non-singularity behavior more on shorter time scales. The smaller component of the accelerometer data will not change the output of the filter from one time step to another, but over long time scales the accelerometer's contribution will help to compensate for the drift of the gyroscope and will bring the filter back to an accurate value of the pitch and roll. This will produce a system that will have the same accuracy as the accelerometer, but have the non-singularity filled range of the gyroscope.

After various attempts, I decided that weighting the gyroscope data with 0.99 and the accelerometer data with 0.01 got the behavior I desired. This can be seen in the first image below when the complementary filter isn't affected by the low-pass accelerometer's odd noise at around 50000 milliseconds, time step. Additionally, in the second image, the complementary filter follows the same shape as the gyroscope, and is euqally as non-noisy, but the accelerometer data is able to bring the filter's output back to an accurate value.

# <img src="Images/Lab 2/complementaryFilterPitchAlpha0.01.png" style="max-width:75%"/>

# <img src="Images/Lab 2/complementaryFilterRollalpha0.01.png" style="max-width:75%"/>

## Sampling Data

### Collection and Storage of Time-Stamped IMU Data

In order to achieve the highest rates of data transfer as possible, I collected the data in arrays on the Artemis Nano, filling the arrays entirely before sending any data over Bluetooth to my laptop. This was done in Arduino by looping over the arrays of fixed size and updating their entries. For example:

```cpp
time_array[counter] = (int)millis();

...

theta_filter_rad[counter] = (theta_filter_rad[counter-1] - dy)*(1.0-alpha_complimentary_filter)+(((theta_a*180)/M_PI)*alpha_complimentary_filter);
phi_filter_rad[counter] = (phi_filter_rad[counter-1] + dx)*(1.0-alpha_complimentary_filter)+(((phi_a*180)/M_PI)*alpha_complimentary_filter);
```

The above code looped through the lists, incrementing `counter` until it reached the length of the list. At this point, the entire list of data was transmitted to my laptop and printed to the Arduino serial monitor one index at a time. Below is how this data was organized for transmission. The "CMD21:" section of the data was used on the Python end of the system in order for it to determine what data was sent with this associated command.

# <img src="Images/Lab 2/10secondGeneratedData.png" style="max-width:50%"/>

The arrays storing the pitch, roll, and time values were 3000 indices long, and all of these data were stored in lists in Python on my laptop after being transmitted by the Artemis Nano.

# <img src="Images/Lab 2/10secondStoredData.png" style="max-width:50%"/>

I believe that it makes most sense to store the pitch, roll, and time data in seperate arrays because this decreases any additional memory that might be required to store an array of specialized structs. Reducing the amount of memory each one of those structs might take means that there is more memory to allocate for the data arrays, allowing them to be larger and to store more data. 

While storing all this different data in one large array might seem like it would save storage, this might not actually be true. Some of the data points, like time, can be stored as integers, while others have to be represented as a float. Therefore, if you stored all the data in one list, you would have to cast variables that require less storage to variables that require more storage. Additionally, storing all of the data in one large array would become harder for the Python scripts to sift through and use, adding to the computational complexity of analyzing the sensor's readings.

These arrays should primarily be of floats and integers. This is because the data coming from the IMU and from the internal timing mechanisms of the Artemis Nano are themselves floats and integers, and it is not worth the time to convert all of this data to a different type of variable in order to store it. I think that it is best to store it as the type of variable it is returned as, and then cast the data to another form of variable if need be later.

As discussed in Lab 1, the amount of memory that I can allocate to my arrays depends on the number of arrays that I have. If, for example, I only care about the pitch and roll output from the complementary filter and the time, then I would only need three arrays. Two of these arrays would be of floats, and the time associated array would be made of integers. This means that to store one element in each of these three arrays will collectively cost 10 bytes of memory, because a float is 4 bytes and an integer is 2. Given that there are 384 kB of RAM, this means that I can make my three arrays almost 39000 indices long.

### Duration of IMU Data Collection

In order to ensure that the robotic system will have enough data to work with at a time, it is helpful to have data be collected for large amounts of time. I was able to get my system to collect data for more than 10 seconds at frequency of about 294 data points per second, as shown below.

# <img src="Images/Lab 2/10secondPitch.png" style="max-width:75%"/>

# <img src="Images/Lab 2/10secondRoll.png" style="max-width:75%"/>

# <img src="Images/Lab 2/10secondText.png" style="max-width:75%"/>

## Record a Stunt

The car is very responsive and very fast. It is able to change directions very quickly, as well as flip over when changing from moving forward to moving backward or vice versa. The button on the top left of the remote seems to be some sort of "go crazy" button that causes the car to move forward for a little bit, do a couple of flips, turn, move forward again, and then repeat this process until any of the buttons are pressed again. I noticed that it is a little bit difficult to turn the car accurately, and I would often overshoot the turning angle that I was trying to achieve. Similarly, it was hard to turn while moving forward or backwards, often resulting in chaotic motions.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kv3FrmjujaU"
  title="ECE 4160: Lab 2 Car Tricks" frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/6KccTDvJOws"
  title="ECE 4160: Lab 2 Spinning Car" frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>


