--- 
title: "Lab 3"
description: "Lab Description and Report"
layout: default
---

# <img src="Images/Lab 3/distance_test_1.jpg" style="max-width:75%"/>
# <img src="Images/Lab 3/distance_test_2.jpg" style="max-width:75%"/>


# Instructions

## Power up your Artemis with a battery!

**1. You will need a JST connector and one of the 650mAh batteries from your RC car.**
Done

**2. Separate the wires on your battery and cut them one at a time. Cutting both wires at the same time will short the terminals and destroy your battery. If you do short your battery, please inform one of the TAs.**
Done

**3. Solder the battery wires to the JST jumper wires. If you are unsure how to do this, please ask one of the TAs.**
Done

**4. Use heat shrink to insulate the exposed portion of wire. Electrical tape can fall off over time and leave a sticky residue on your robot.**
Insert Image of Battery

# <img src="Images/Lab 3/battery_in_board.jpg" style="max-width:75%"/>

**5. Check the polarity of these wires (positive on the battery should be connected to the positive terminal on your Artemis – this may mean that you need to connect opposite color wires).**
The polarity was actually oposite to how the board wanted it, so I soldered black to red and red to black. (refer to image)

**6. Power up your Artemis using only the battery and no connection through the USB C port. Try sending BLE messages back and forth from your laptop to ensure that the Artemis is powered on correctly and can send messages completely untethered.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/IdEWK477Lec" title="ECE 4160: Lab 3 Remote Connection" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 
   
## Using the Arduino library manager, install the SparkFun VL53L1X 4m laser distance sensor library.
Done

## Connect the QWIIC break-out board to the Artemis
Done

## Connect the first ToF sensor to the QWIIC breakout board.
**1. You will have to cut one end of a QWIIC cable and solder the other to your sensor. You have two long cables and two short ones, choose wisely.**
Done. Discuss choice for wire lengths, add image of setup next to the car

# <img src="Images/Lab 3/sensor_layout.jpg" style="max-width:75%"/>
# <img src="Images/Lab 3/sensor_layout_with_car.jpg" style="max-width:75%"/>

**2. Think about which color attaches to SDA/SCL?**
blue = sda, yellow = scl

## Scan the I2C channel to find the sensor
**1. Go to File->Examples->Apollo3->Wire and open Example05_wire_I2C**
Done

**2. Browse through the code to see how to use i2c commands.**
Done

**3. Run the code. Does the address match what you expected? If not, explain why.**
This does not match what I expected. In the documentation for this sensor, it says that the default address of the ToF is 0x52, but the example code is returning 0x29. These are very different addresses, resulting in decimal values of 82 and 41, respectively.

However, in binary, these addresses look more similar. In binary, 0x52 becomes 0b1010010, and 0x29 becomes 0b101001. These are very similar, with 0x29 being 0x52 shifted one bit to the right. This makes sense when looking more carefully in the documentation of the sensor, where it says "If the least significant bit is low (that is, 0x52) the message is a
master-write-to-the-slave. If the LSB is set (that is, 0x53) then the message is a masterread-from-the-slave". This means that the least significant bit just determines the action that the board does with the sensor, and that the rest of the bits are the true address. Therefore, this address is what is expected.

## ToF sensor modes
**The ToF sensor has three modes (Short, Medium, and Long) that optimize the ranging performance given the maximum expected range. Discuss the pros/cons of each mode, and think about which one could work on the final robot. (Note: medium mode is only available with the Polulu VL53L1X Library).**

**<pre> .setDistanceModeShort(); //1.3m .setDistanceModeMedium(); //3m .setDistanceModeLong(); //4m, Default </pre>**

Short:

### Pros
1. It can be more accurate and precise --> less error

### Cons
1. It doesn't give as much range, so the system would have to adapt more quickly
2. System might not be able to adapt quick enough to incoming information depending on how fast the car is moving

Long:

### Pros
1. It can give a larger range of data --> good for developing a wider field of view in order to understand more of the car's environment

### Cons
1. Can be less accurate and prone to more error

Medium:

### Pros
1. It has a longer range than short
2. It can be more accurate than long

### Cons
1. It isn't as accurate as short
2. It doesn't have as long of a range as long

Our robot is supposed to be fast. The actual car can be very fast, stoping and turning very quickly as a response into the input of the system. Therefore, I do not beleive that the car will need a larger range of data in advance in order to avoid obstacles, so I think that I will use the short sensing mode.

## Test your chosen mode
**1. Use the “..\Arduino\libraries\SparkFun_VL53L1X_4m_Laser_Distance_Sensor\examples\Example1_ReadDistance” example**
**2. Document your ToF sensor range, accuracy, repeatability, and ranging time**


## Connect to Prelab
**Using notes from the pre-lab, hook up both ToF sensors simultaneously and demonstrate that both work.**

In order to use both ToF sensors at the same time, there are two options. 
1. We change one of the addresses of the ToF sensors so that they have different ones. To do this, use the Xshut pin on the sensor whose addres you're not changing (to turn it "off"), and then use the set.I2CAddress() function from the Sparkfun library to change the address (the default address of the Tof is 0x52/0x53 (for read/write), and the default addres of the I2C is 0x69, so I would just need to pick an address that is different than those. Then I can turn on the other ToF sensor, and it has the default address. I am not sure how this would affect the I2C address of the IMU device, but I think it will be okay)
2. We ignore the data from one of the ToF sensors whenever we want data from the other, by turning the one we don't want "off" (turning on the Xshut pin). This might be easier programming wise, but I like the other way better.

I chose the first method. It works well. I haven't tried it with the IMU as well yet.

## Speed it up
**In future labs, it is essential that the code executes quickly, therefore you cannot let your code hang while it waits for the sensor to finish a measurement. Write a piece of code that prints the Artemis clock to the Serial as fast as possible, continuously, and prints new ToF sensor data from both sensors only when available.**

**1. The distanceSensor.checkForDataReady() routine can be called to check when new data is available.**

Done.

**2. How fast does your loop execute, and what is the current limiting factor?**

Based on the times reported at the end of each void loop() iteration, my for loop ran at about 700 Hz (from data collected during experimentation). The time of flight sensors, however, ran much slower. Both time of flight sensors only updated six times in a 407 millisecond period, averaging at about 14.7 Hz when both sensors run simultaneously. Currently, the limiting factor is how long it takes for the ToF sensors to be ready to give new data measurements.

## Edit Lab 1 Stuff
**Finally, edit your work from Lab 1, such that you can record time-stamped ToF data and IMU data for a set period of time, and then send it over Bluetooth to your computer.**

Done.

## Include a plot of the ToF data against time.

## Include a plot of the IMU data against time.




# Write-up

## Prelab
**1. Note the I2C sensor address**

There are different types of communication protocols between sensors and microcontrollers like the Artemis Nano, but a major one is the I2C protocol. In I2C, all sensing devices share a serial clock line, a serial data line, power, and ground. All of the connected devices send and receive data from the controller along the shared data line, which greatly decreases the number of physical connections the sensors need to make with the microcontroller, as opposed to a SPI communication protocol where every sensor needs its own communication lines.

Due to the fact that many devices can share one I2C bus, each of these devices must be uniquely addressible by the microcontroller. This allows the contorller to indicate which device it wants to write/read data to/from, and then that particular device can acknowledge that it is ready to receive/send data. Then communication between the controller and device can take place. However, it is common to have multiple sensors with the same address. For example, in this lab, the IMU has an I2C address of 0x69.

Interestingly, while the documentation for the Time of Flight, ToF, sensors says that they have an I2C address of 0x52/0x53, their actual defualt address is 0x29. This will be explained in the "**insert section title here**" section below. Both ToF sensors sharing the same address presents an issue when wanting to create a system that uses them concurrently. Solutions to this problem are discussed below.

**2. Briefly discuss the approach to using 2 ToF sensors**

The Time of Flight, ToF, sensors, communicate with the Artemis board through an I2C communication protocol. As mentioned above, in order for the microcontroller to communicate with all powered, connected devices, each sensor must have a unique I2C address. This becomes an issue when using two ToF sensors that have the default address, and there are two ways of overcoming this communication obstacle.

The first method to provide stable communication to both of the ToF sensors doesn't require the devices to have unique addresses. Devices only need to have unique addresses on an I2C bus if the controller is trying to communicate with said device, and this communication can only take place if the that sensor is also powered. Therefore, if the controller powered down one of the sensors with duplicate addresses when trying to read from the other sensor, there will be no issues with addressing. Then, after collecting the data of interest, the contorller can turn both devices on until it has to read from one of them again, at which time it would turn the other device off while taking the measurement.

The second method only requires you to turn a sensor off once. During the setup of the program, if one of the sensors are turned off and can't communicate with the controller, then the program can actually chagne the address of the device that it can communicate with. By changing the addresses of one of the ToF sensors, there will no longer be an address issue, and both devices can continuously be powered through the duration of the program.

I chose to implement the second solution, as it only required a little bit of computation at the setup of the program to fix the communication issue with the I2C devices, as opposed to constantly needed to turn devices off and on at a high pace.

In order to turn the second ToF sensor off while changing the address of the first, I set the XSHUT pin on the second ToF sensor high. This pin places the sensor into hardware-shutdown, making it "invisible" to the controller, and now there is only one device with the original duplicated address alive on the I2C bus. This allows me to set a new address for the sensor the first sensor, and this new address should be different than the IMU address and the default ToF address. This is shown in the code snippet below:

```cpp
digitalWrite(SHUTDOWN_PIN, LOW); // turns ToF sensor 1 off
int add;
add = distanceSensor0.getI2CAddress(); // gets the original I2C address
Serial.print("The old address of sensor 0 is: ");
Serial.println(add);
distanceSensor0.setI2CAddress(0x40); // sets a new I2C address
add = distanceSensor0.getI2CAddress();
Serial.print("The new address of sensor 0 is: ");
Serial.println(add);
digitalWrite(SHUTDOWN_PIN, HIGH); // turns ToF sensor 1 back on
```

**3. Briefly discuss placement of sensors on robot and scenarios where you will miss obstacles**

In our lab kit, we were given four QWIIC connectors, two of them almost twice the length of the other two. This prompted me think about how the sensors would be laid out on the robot befor cutting the end off of two of the connectors and soldering them to the ToF sensors, which do not have QWIIC connect ports. With an understanding that the robot car has to avoid obstacles that are approaching it from the front, and at some point will have to maintain a specified distance from a wall to its side while driving forward, I thought that the optimal placements of the ToF sensors would be on the front and side. 

I thought that the accelerometer should be as close to the center of the robot as possible, since the car implements differential drive, and this would be the center of the car's rotation. The picture below shows a sensor layout next to the robot car to illustrate their relative sizes. The two green sensors are the ToF sensors. I decided to connect one of the ToF sensors to a longer QWIIC connector and the other to a short one so that the ToF sensor in the front of the car has a farther reach. The ToF sensor on the side of the car is closer to the center of the car, where the IMU will also be, so both of those sensors can use the shorter QWIIC connectors.

# <img src="Images/Lab 3/sensor_layout_with_car.jpg" style="max-width:75%"/>

**4. Sketch of wiring diagram (with brief explanation if you want)**


## Lab Tasks


**1. Picture of your ToF sensor connected to your QWIIC breakout board**


**2. Screenshot of Artemis scanning for I2C device (and discussion on I2C address)**

The documentation for the ToF sensor indicates that default I2C address is either 0x52 or 0x53, depending on if the controller is writing or reading from sensor. However, when scanning for I2C devices connected to the Artemis board, the program reported that the address of the ToF sensor was actually 0x29. These are very different addresses, and at first glance they do not look remotely similary in neither hexidecimal nor decimal.

However, when these addresses are represented in binary, they look more similar. In binary, 0x52 becomes 0b1010010, 0x53 becomes 0b1010011, and 0x29 becomes 0b101001. These are very similar, with 0x29 being 0x52/0x53 shifted one bit to the right. While this might initially seem odd, it makes sense when thinking about these numbers as digital addresses and commands used on the I2C bus. The controller outputs a sequence of binary signals in order to indicate what device that it would like to communicate with on the bus. This would 0b101001. But then why would the addresses be reported as 0x52/0x53 in the documentation?

By further looking into the sensor's documentation, it can be seen that if the least significant bit of the address is low, then the controller is indicating that it wants to write to the sensor. If the least significant bit is high, then the controller wants to read from the sensor. But the true address of the ToF sensor is 0x29, and this can't shouldn't automatically change depending on if the controlelr is executing a read or write command. Therefore, it seems that the true address of the sensor was shifted left by one, creating a new least significant bit that did not contain address information, and could be changed to indicate how the controller wants to interact with the sensor. 

**add screenshots??**

**3. Discussion and pictures of sensor data with chosen mode**

The ToF sensor comes with two available sensing methods: short distance mode and long distance mode. The short distance method focuses on accurately measuring distance up to 1.3 meters away from the sensor, and the long distance mode tries to accurately measure up to 4 meters away. Whichever mode the sensors are placed in is completely up to the programmer, creating a need to compare their benefits and costs.

The short distance sensing mode can be more accurate, as it is discretizing a smaller allowable range of data. However, this accuracy is only applicable for distances close to the sensor, meaning that the sensor may miss important data coming from obstacles or objects that are outside of its range of accurate detecting. This can causes systems using the short distance method to not be able to plan much in advance to avoid obstacles, and they will need to be able to adapt to data faster as they will have to get physically closer to objects to accurately detect them.

The long distance sensing mode has a larger range, allowing for systems using it to more quickly react to events that are farthe away from it. This could be good for the car robot developed in this lab because it would allow it to detect obstacles or walls sooner. However, this increased range corresponds with a less accurate measurement, and any increased error in measurement could have negative effects on the state estimation and controls of the robot. Additionally, ranging over a larger distance means that it would take more time to make measurements, slowing down the sampling rate.

Since the goal of this class is to develop a fast robot car, speed and accuracy is paramount. The car has very fast dynamics, being able to change heading, direction, and speed incredibly quickly. Therefore, this system should be quick enough to adapt to rapibly incoming data about its environment, so I will use the short distance measurement mode for the ToF sensors. While this mode is only optimal in a smaller range of distances, it is very accurate and also faster than the long distance mode, allowing for the system to get more reliable data.

To test the accuracy and range of the ToF sensor when operating in the short distance mode, I created an experiment that took many ToF measurements at various distances from a wall and then looked at the results of these measurements. To do this, I taped a single ToF sensor to the front of a box, such that it was perpendicular to the floor and was pointing directly at the wall in front of the box. Using a tape measure, I placed the box with the ToF sensor taped to it and various, controlled distances from the wall. Once the box had been placed in position, I took thousands of ToF sensor readings without moving the box. I averaged these values, found their standard deviation, moved the box a little bit further away from the wall, and repeated the whole process from 0 m to 2.5 m away from the wall in 0.1 m intervals.

The first two graphs below depict the average measured data from the ToF sensor at various fixed distances from a wall, as well as the error between the true distance from the wall and the measured distance. Notice how the accuracy of the sensor dramaticaly decreases past the maximum ranging point for the short distance method, which is 1.3 m.

# <img src="Images/Lab 3/tof_data.png" style="max-width:75%"/>
# <img src="Images/Lab 3/tof_error.png" style="max-width:75%"/>

These two graphs show the same data as above, but zoomed in on the operating range of the ToF sensor when in the short distance measuring mode.

# <img src="Images/Lab 3/tot_tof_data.png" style="max-width:75%"/>
# <img src="Images/Lab 3/tot_tof_error.png" style="max-width:75%"/>


**4. Tof sensor speed: Discussion on speed and limiting factor; include code snippet of how you do this** 

Based on the times reported at the end of each void loop() iteration, my for loop ran at about 700 Hz (from data collected during experimentation). The time of flight sensors, however, ran much slower. Both time of flight sensors only updated six times in a 407 millisecond period, averaging at about 14.7 Hz when both sensors run simultaneously. Currently, the limiting factor is how long it takes for the ToF sensors to be ready to give new data measurements.

**Include some of the data collected, it is in the Notes app**

# <img src="Images/Lab 3/tof_with_stopRanging.png" style="max-width:75%"/>
# <img src="Images/Lab 3/tof_without_stopRanging.png" style="max-width:75%"/>

**5. 2 ToF sensors and the IMU: Discussion and screenshot/video of sensors working in parallel**




**6. Time v Distance: Include graph of data sent over bluetooth (2 sensors)**


**7. Time v Angle: Include graph of data sent over bluetooth**



This is not a strict requirement, but may be helpful in understanding what should be included in your webpage. It also helps with the flow of your report to show your understanding to the lab graders.


