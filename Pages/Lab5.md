--- 
title: "Lab 5"
description: "Lab Description and Report"
layout: default
---

# Lab 5: Linear PID Control and Linear Interpolation

**General Information Images**

# <img src="Images/Lab 5/loop_speed_with_extrapolation.png" style="max-width:90%"/>
# <img src="Images/Lab 5/loop_speed_without_extrapolation.png" style="max-width:90%"/>
# <img src="Images/Lab 5/rawDistance_Extrapolated_lpf.png" style="max-width:90%"/>
# <img src="Images/Lab 5/rawDistance_Extrapolated.png" style="max-width:90%"/>

**P Only**

<iframe width="560" height="315" src="https://www.youtube.com/embed/biKzvUNXVKs" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

<iframe width="560" height="315" src="https://www.youtube.com/embed/wIy1Zj0XlH4" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="Images/Lab 5/Just P/d_term.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/no_wall_terms.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Just P/overshoot_terms.png" style="max-width:90%"/>

**P and D**

<iframe width="560" height="315" src="https://www.youtube.com/embed/WuX_4CsHXg0" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="Images/Lab 5/P and D/less_overshoot_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/P and D/less_overshoot_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/P and D/less_overshoot_terms.png" style="max-width:90%"/>

**Larger P and D**

<iframe width="560" height="315" src="https://www.youtube.com/embed/e54BBLMmohk0" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="Images/Lab 5/Larger P and D/constants.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Larger P and D/final_distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Larger P and D/final_pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Larger P and D/final_terms.png" style="max-width:90%"/>

**Extrapolated**

<iframe width="560" height="315" src="https://www.youtube.com/embed/5x_ZpusBX4U" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

# <img src="Images/Lab 5/Extrapolated/constants.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/distance.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/pwm.png" style="max-width:90%"/>
# <img src="Images/Lab 5/Extrapolated/terms.png" style="max-width:90%"/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/Gream7-C52o" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

## Prelab

### Clearly describe how you handle sending and receiving data over Bluetooth

In order to efficiently tune the PID parameters, I had to restructure how the Artemis handles Bluetooth commands sent from my laptop. Originally, I sent a singular command from my laptop that set all of the values of the PID parameters, determined how long to run the PID control food, and indicated whether or not the distance data should be extrapolated. This command looked like:

```python
goal_distance = 0.25 # 500 mm ~ 20 inches
kp = 0.12
ki = 0.0
kd = 0.0 #0.007, 0.3
lower_limit = 40
loop_duration = 5 # seconds
extrapolate = 1

string = str(goal_distance) + "|" + str(kp) + "|" + str(ki) + "|" + str(kd) + "|" + str(lower_limit) + "|" + str(loop_duration) + "|" + str(extrapolate)

ble.send_command(CMD.DISTANCE_PID, string)
```

The Artemis would receive this command, set the appropriate PID values, and then run the controller continuously for a specified amount of time, regardless of if the goal was achieved. 

After using this method for a couple of tries, I realized that the system could be more adaptable if I separated these functions into different Bluetooth commands. For example, I created a `SET_DIST_PID_PARAMS` that allowed me to change the PID constants and the desired goal at any point in time, even if the PID control loop was already running. `TURN_ON_MOTORS` and `TURN_OFF_MOTORS` allowed for the motors to be driven or turned off, respectively, so that I could run the PID loop with and without the car moving during the test period. The command `DISTANCE_PID` started the PID control, which ran until the Artemis received a `STOP_PID` command. 

While the PID loop is running, arrays of relevant data are populated so long as they are not full, like so:


```cpp
if ( counter < length_of_array ){
           yaw1[counter] = angle;
           P[counter] = proportional_term_a;
           I[counter] = integral_term_a;
           D[counter] = derivative_term_a;
           pwm[counter] = pwm_input_a;
           time_array[counter] = current_time;
           counter = counter + 1;
       }
```


The Artemis then sent all available data to my laptop when it received a `SEND_DISTANCE_PID_DATA` command.


This modularity gave me greater control over the robotic system, allowing me to actively change PID parameters while the control loop was running and also easily change how long I ran the control loop for.


 In Python, this looked like:


```python


goal_distance = 500 # 500 mm = 19.7 inches
kp = 0.27
kd = 0.11
extrapolate = 1
lpf_distance_alpha = 0.08


string = str(goal_distance) + "|" + str(kp) + "|" + str(kd) + "|" + str(extrapolate) + "|" + str(lpf_distance_alpha)


ble.send_command(CMD.SET_DIST_PID_PARAMS, string)


ble.send_command(CMD.TURN_ON_MOTORS, "")


# ble.send_command(CMD.TURN_OFF_MOTORS, "")


# wait some time until I’m ready to start the control loop


ble.send_command(CMD.DISTANCE_PID, "") # Starts the distance PID control loop


# run the loop for as long as I would like


ble.send_command(CMD.STOP_PID, "") # Stops the distance PID control loop


# can then send data from the Artemis to laptop, if wished


# removes any old data in the global data arrays that are populated when the “SEND_DISTANCE_PID_DATA” command is called
tof0 = []
time_array = []
P = []
D = []
pwm = []


ble.send_command(CMD.SEND_DISTANCE_PID_DATA, "")
```

The Artemis board continuously listened for all of these commands. If one of the commands were received, then a flag associated with that command was changed to create the desired behavior on the Artemis. For example:

```cpp


       case DISTANCE_PID:


           distance_pid = 1;
           counter = 0;
           Serial.println("Started distance PID");
           break;


       case STOP_PID:


           distance_pid = 0;
           angular_pid = 0;
           turn_off_motors = 1;
           distance_pid_running = 0;
           Serial.println("Stop PID.");
           break;


       case SEND_DISTANCE_PID_DATA:
           send_distance_pid = 1;
           Serial.println("Sending distance PID data");


           break;
```

In the `void loop()`, the value of these flags could trigger events, such as when to run the PID loop or when to send the PID data. In order to achieve this behavior, I made PID controls and data transmissions into functions that could be called if a certain set of flags were set to true. For example, here is a shortened version of some of my `while` loop in the `void loop()`:

```cpp
// While central is connected
       while (central.connected()) {


           // Send data over Bluetooth
           write_data();


           // Read data from Bluetooth
           read_data();


           # if distance_pid is true, run the PID control loop


           # if send_distance_pid is true, send the collected data


           # if turn_off_motors is true, stop the motors from running
       }
```

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


