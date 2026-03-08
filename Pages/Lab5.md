--- 
title: "Lab 5"
description: "Lab Description and Report"
layout: default
---

# Lab 5: Linear PID Controll and Linear Interpolation

# <img src="Images/Lab 4/wiring.png" style="max-width:90%"/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/Gream7-C52o" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

## Prelab
### Clearly describe how you handle sending and receiving data over Bluetooth
### Consider adding code snippets as necessary to showcase how you implemented this on Arduino and Python
## Lab Tasks
### P/I/D discussion (Kp/Ki/Kd values chosen, why you chose a combination of controllers, etc.)
### Range/Sampling time discussion

```cpp
void
loop()
{
    // Listen for connections
    BLEDevice central = BLE.central();


    distanceSensor0.startRanging();
    counter = 0;
    currentMillis = millis();
    loop_stop_time = currentMillis + 60*1000;

    while(millis() <= loop_stop_time){

        if (distanceSensor0.checkForDataReady()) {
            distance = distanceSensor0.getDistance(); //Get the result of the measurement from the sensor
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

`60 seconds, 1957 reads, 32 frequency`

### Graphs, code, videos, images, discussion of reaching task goal
### Graph data should include Tof vs time and Motor input vs time (and whatever helps with debugging)

