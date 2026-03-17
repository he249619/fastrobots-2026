--- 
title: "Lab 6"
description: "Lab Description and Report"
layout: default
---

# Lab 6: Orientation Control

## Prelab

To control the orientation of the car, I had to implement a few more Bluetooth commands that the Artemis could use for the logic associated with the orientation PID controller I implemented. 

In order to quickly iterate through PID constants, I created a `SET_ANG_PID_PARAMS` command that set the values of `kp`, `kd`, and `ki`, even if the control loop was already running. To start the orientation control loop, I sent the command `START_ANGULAR_PID`, and `SEND_ANGULAR_PID_DATA` sent any data collected while the control loop ran back to my laptop for analysis and visualization. 

Similarly to Lab 5, the code on the Artemis board then raised or lowered different flags, depending on what commands were received, in the `void loop()`. Below is some pseudocode describing my updated process of checking on the global flags altered by the Bluetooth commands:

```cpp
while there is a Bluetooth connection {


	Read any incoming Bluetooth commands
	Send any outgoing data over Bluetooth


	if the flag associated with running the distance PID loop
	is set high by the Bluetooth commands:
		run one iteration of the distance PID loop


	if the flag associated with sending the data collected 
while the distance PID loop ran is set high by 
the Bluetooth commands:
		send all relevant data


	if the flag associated with running the angular PID loop
	is set high by the Bluetooth commands:
		run one iteration of the angular PID loop


	if the flag associated with sending the data collected 
while the angular PID loop ran is set high by 
the Bluetooth commands:
		send all relevant data


	if the flag associated with cutting power to the
motors is set high by the Bluetooth commands:
		stop the motors
}
```

## Lab Tasks

### PID Input Signal

In order to do control on the orientation of the robot, it was necessary to use the IMU to collect data of the yaw of the robot. Unfortunately, it isn’t possible to get yaw from the accelerometer alone, so the gyroscope is needed. The issue with the gyroscope is that, while it has very little noise compared to the accelerometer, it drifts farther away from the true value over time. In order to get yaw, the gyroscope measurement would have to be integrated over time, further increasing the negative effects of the overall drift. 

The accumulation of drift could be somewhat avoided by subtracting the offset that the gyroscope reports when the IMU is not rotating. This offset, or bias, is what causes the gyroscope's data to veer from the true yaw of the robot. While this may work great on short time scales, the gyroscope’s bias may change over time, so it is not a good long term solution.

The IMU has a built-in alternative option: a Digital Motion Processor, or DMP. The DMP uses the IMU’s accelerometer, gyroscope, and compass to perform motion processing algorithms on the IMU chip itself, while simultaneously running background calibrations . This data can be sent to the Artemis in the form of a quaternion, which then only needs to be transformed into Euler angles in order to get the yaw of the robot car. 

With the DMP implemented, only two questions of practicality remain: can the gyroscope measure angular velocities that the car will rotate at, and can the DMP return orientation information fast enough for us to use? Luckily, the gyroscope has a programmable range of measurements, and when the DMP operates it can register angular velocities between -2000 degrees per second and 2000 degrees per second, which is far beyond the speed at which the car will go. Additionally, the DMP’s output data rate can be set when the IMU is initialized:

```cpp
// Set the DMP output data rate (ODR): value = (DMP running rate / ODR ) - 1
// E.g. for a 5Hz ODR rate when DMP is running at 55Hz, value = (55/5) - 1 = 10.
success &= (myICM.setDMPODRrate(DMP_ODR_Reg_Quat6, 1) == ICM_20948_Stat_Ok);
```
I decided to set the output data rate to about 25 Hz as an initial test of the system. However, this did not bottleneck my control loop, which ran at about 190 Hz. I achieved this by holding the angle value constant whenever no new data was available. For a more adaptive interpolation method, I could have linearly extrapolated from the two most recent sensor readings like in Lab 5. I found that this was not necessary to achieve a low settling time, but this could be useful for future iterations. 

### Proportional Discussion

I first implemented the proportional logic in the PID controller. Recognizing that the maximum error I could get between the car’s current heading and the goal heading was 180°, I decided that I should first try a `kp` value of 1, as this would cause the maximum PWM signal to be 180. While this might seem large when compared to the PWM signals of Lab 5, this signal is being used for a completely different motion. Rotational motion requires more energy, and the deadband is significantly higher. Here, I set the deadband to 160, which is higher than what I found in Lab 3, but this allows for the car to move more smoothly. 

In order to produce rotational movement, I set one set of wheels to drive forward with input PWM, and the other set of wheels to drive backward with the same PWM. If the PWM were to be negative, the direction of rotation would switch.

Using `kp = 1.0` and `kd = ki = 0.0` was a good start, but it resulted in a significant amount of overshoot. The following data was collected when the car was attempting to rotate 45° to the right.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Osg98f0OM9E" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="IImages/Lab 6/Just P/angle.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P/constants.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P/p.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P/pwm.png" style="max-width:90%"/>

When the car attempted to rotate 180°, it overshoot even more.

<iframe width="560" height="315" src="https://www.youtube.com/embed/NP9hRd67f1Q" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="IImages/Lab 6/Just P, Overshoot/angle.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P, Overshoot/constants.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P, Overshoot/p.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/Just P, Overshoot/pwm.png" style="max-width:90%"/>

### Proportional and Derivative Discussion

To account for this overshoot, I needed to implement derivative control. 

If I had not used the DMP, then calculating the derivative of the yaw would be straightforward since the gyroscope outputs the rate of change of rotation. However, with the DMP in place, I was not able to access the raw acceleration and gyroscope readings. I believe that this is due to the IMU prioritizing using its on-chip computation to run the DMP.

Normally, I would not take the derivative of the integral of a noisy and biased signal, but the DMP’s internal sensor calibration causes its output to be more accurate than a filter that I might have created.

To prevent any derivative kick that might occur when initializing the system or changing the goal heading, I take the derivative of the heading instead of the error. This was a little bit challenging to implement since I decided to wrap my angles between [180,° -180°), but it just required a little bit more math to make it work. Additionally, I low-pass filtered the derivative of the heading in order to remove noise caused by the car jittering back and forth.

After a lot of trials, my final values for `kp` and `kd` were `1.5` and `0.03`, respectively.

<iframe width="560" height="315" src="https://www.youtube.com/embed/EXq_kSRCReA" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="IImages/Lab 6/OG P and D/angle.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/OG P and D/constants.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/OG P and D/p.png" style="max-width:90%"/>
# <img src="IImages/Lab 6/OG P and D/pwm.png" style="max-width:90%"/>

The following shows how the setpoint can be changed as the controller is running, as well as how the car can adapt to disturbances. Interestingly, the behavior of the car here is slightly less ideal than that of the previous video. I believe that this is a result of a decrease in charge of the motor battery.

<iframe width="560" height="315" src="https://www.youtube.com/embed/uncqa34D-34" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 


### Integral Discussion

Similar to Lab 5, I found that the system did not encounter any steady state error, so I did not implement an integral portion in the control logic.

