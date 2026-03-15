--- 
title: "Lab 5"
description: "Lab Description and Report"
layout: default
---

# Lab 5: Linear PID Control and Linear Interpolation

**General Information Imgaes**

# <img src="Images/Lab 5/loop_speed_with_extrapolation.png" style="max-width:90%"/>
# <img src="Images/Lab 5/loop_speed_without_extrapolation.png" style="max-width:90%"/>
# <img src="Images/Lab 5/rawDistance_Extrapolated_lpf.png" style="max-width:90%"/>
# <img src="Images/Lab 5/rawDistance_Extrapolated.png" style="max-width:90%"/>

**P Only**

# <img src="Images/Lab 5/Just P/d_term.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_terms.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_terms.png" style="max-width:90%"/>

**P and D**

# <img src="Images/Lab 5/P and D/less_overshoot_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/P and D/less_overshoot_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/P and D/less_overshoot_terms.png" style="max-width:90%"/>

**Extrapolated P and D**

# <img src="Images/Lab 5/Extrapolated P and D/constants.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated P and D/final_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated P and D/final_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated P and D/final_terms.png" style="max-width:90%"/>

**Extrapolated**

# <img src="Images/Lab 5/Extrapolated/constants.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/terms.png" style="max-width:90%"/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/Gream7-C52o" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

## Prelab

### Clearly describe how you handle sending and receiving data over Bluetooth

### Consider adding code snippets as necessary to showcase how you implemented this on Arduino and Python

## Lab Tasks

### P/I/D discussion (Kp/Ki/Kd values chosen, why you chose a combination of controllers, etc.)

### Range/Sampling time discussion

In order to determine the sampling frequency of the ToF sensor when in the short distance sensing mode, I implemented the following loop on the Artemis board. This creates a loop that runs for 60 seconds, counts the number of times that the ToF sensor is able to read data within that time interval, and from that data calculates the sampling rate.

```cpp
void
loop()
{
    distanceSensor0.startRanging();
    counter = 0;
    currentMillis = millis();
    loop_stop_time = currentMillis + 60*1000;

    while(millis() <= loop_stop_time){

        if (distanceSensor0.checkForDataReady()) {
            distance = distanceSensor0.getDistance();            
distanceSensor0.clearInterrupt();
            distanceSensor0.stopRanging();
            distanceSensor0.startRanging();
            counter = counter + 1;
        }
    }
    Serial.print((loop_stop_time - currentMillis)/1000);
    Serial.print(" seconds, ");
    Serial.print(counter);
    Serial.print(" reads, ");
    Serial.print(counter/((loop_stop_time - currentMillis)/1000));
    Serial.println(" frequency");
}

```
On average, this loop indicated that the sampling rate of the ToF sensor was about 33 Hz. The maximum sampling frequency of the ToF is 50 Hz when in the short distance sensing mode, demonstrating that my system is within the expected range but not operating ideally.

Even if the sensor performed at 50 Hz, running a control algorithm at this frequency would not allow for the robotic car to adapt quickly enough to its environment. To increase the frequency of the control loop, we can estimate the state of the data whenever the ToF sensor is not able to provide an updated sensor reading.

### Decoupling Sensor and Control Frequency

To increase the frequency of the control loop, we can estimate the state of the data whenever the ToF sensor is not able to provide an updated sensor reading. 

A basic estimation would be assuming that the distance from the car and the object in front of it in between sensor readings does not change. While this is a terrible assumption, it allows the controller to run faster.

*add code, graphs, and discussion about this*

A more accurate method of estimating data in between sensor readings would be to extrapolate the sensor data based on the two most recent points, and use this to make a guess about where the car is relative to an obstacle in front of it. This makes a prediction of what the sensor would read, assuming that the velocity of the car remains the same in between sensor reads. While this may still not accurately illustrate the true state, it is an improvement from the previous assumption.

*add code, graphs, and discussion about this*

### Graphs, code, videos, images, discussion of reaching task goal

### Graph data should include Tof vs time and Motor input vs time (and whatever helps with debugging)


