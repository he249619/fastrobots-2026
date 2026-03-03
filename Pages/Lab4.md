--- 
title: "Lab 4"
description: "Lab Description and Report"
layout: default
---

# Lab 4: Motors and Open Loop Control
## Prelab

### Wiring Discussion

This lab focuses on integrating two DRV8833 dual motor drivers into our system so that we have greater control over how we drive the car as opposed to using the car’s original control PCB. Each of these motor drivers has the potential to drive two motors, but in order to increase the current supply to either motor, both channels on a singular motor driver were combined. This allowed for the motors to produce a larger torque. A diagram of this circuit is shown below.

# <img src="Images/Lab 4/wiring.png" style="max-width:75%"/>

As can be seen in the diagram above, four pins are required to control these two motor drivers. This is because each of the combined channels, AIN1 + BIN1 and AIN2 + BIN2 on both motor drivers, requires an independent Pulse Width Modulation (PWM) signal, which in turn drives the motors of the car. For the first motor driver, I use Pin 9 for the IN1 channel and Pin 11 for the IN2 channel. For the second motor driver, I use Pin 12 for the IN1 channel and Pin 14 for the IN2 channel. The outputs of both motor drivers, AOUT1 + BOUT1 and AOUT2 + BOUT2, got to their corresponding motor’s power and ground, allowing the Artemis to control the motors through the dual motor drivers.

I chose to use these pins as they are on the same side of the Artemis board and more towards the lower half of the board. This allowed me to maneuver the board more freely because all the motor driver connections were centered around one corner.

Importantly, the diagram above illustrates that a Li-Ion 3.7V 850mAh battery is used to power the motors, which has slightly more capacity than the Li-Ion 3.7V 750mAh battery used to power the Artemis. It is vital to separate these two sources of power because the motors can draw a large amount of current, and this current could potentially cause a lot of EMI for the microcontroller if they were looped together into the same power circuit. By separating these components, we are attempting to reduce the EMI exposure of the microcontroller.

## Lab Work

### Picture of your setup with power supply and oscilloscope hookup

Before soldering the battery to the motor drivers, I needed to ensure that they were outputting the expected signal when given a certain input from the Artemis board. In order to do this, I tested each of the motor drivers by connecting them to a DC power supply and monitoring their output with an oscilloscope. I set the power supply 3.7V to simulate the voltage of the Li-Ion battery that will power the motors in the final configuration. This lab setup is pictured below.

You may notice that one of my input pins to the motor controller shown is not one of the pins that I described in the previous section. This is simply because I wanted to test different pin connections, as I found that some of the pins automatically go high when the Artemis board boots up. Pin 9 is one of these pins, and through this experimentation I found out that I have to set it low before in the `setup()` portion of my code in order to change the PWM signal it outputs later. 

# <img src="Images/Lab 4/lab_setup.jpg" style="max-width:75%"/>

<!-- # <img src="Images/Lab 4/closer_up_setup.jpg" style="max-width:75%"/> -->

### Power supply setting discussion (done in previous parts)

### Testing PWM Output (Include the code snippet for your analogWrite code that tests the motor drivers)

When testing to see if the motor drivers produce the correct output, I used the `analogWrite()` function in Arduino to control the PWM output of Pins 9, 11, 12, and 14. For example, with the code shown below, I tested if the motor controllers were able to output the correct duty cycles of about 79% (200 out of the 255 maximum) and 40 percent (100 out of the 255 maximum).

`cpp
 analogWriteResolution(8);

 analogWrite(9, 0);
 analogWrite(11, 100);
`

For each motor driver, I commanded one of the pins associated with it to output either 100 or 200 as the PWM signal and set the other pin as 0, and then tested the opposite as well. This was done with very similar code to that shown above, just requiring me to change the pin numbers and PWM value in the function call.

The output of these tests can be seen below. The first image shows the motor driver output when given an input of 100 out of 255, and the second image shows the output from a 200 out of 255 input. 

# <img src="Images/Lab 4/100_PWM.jpg" style="max-width:75%"/>
# <img src="Images/Lab 4/200_PWM.jpg" style="max-width:75%"/>

It is interesting to note that the Artemis can output PWM signals with different resolutions. When the PWM resolution is set to 8, as I did here, PWM signals can be between 0 and 255. When it is set to 16, however, PWM signals can be set anywhere between 0 and 65535. Resolution this high doesn’t seem necessary for the robot in this lab, but it’s good to know that the option is there if I want to develop very fine tuned inputs into the system later on. 

### Short video of wheels spinning as expected (including code snippet it’s running on)

After ensuring that the motor drivers were outputting the expected values, I each of them to one of the car’s motors one at a time. The motor drivers, and the motors as well, were still powered by the power supply at this point. I tested the behavior of the motors when given the following command continuously:

`cpp
 analogWrite(9, 100);
 analogWrite(11, 0);


 delay(2000);


 analogWrite(9, 0);
 analogWrite(11, 0);


 delay(2000);


 analogWrite(9, 0);
 analogWrite(11, 200);


 delay(2000);


 analogWrite(9, 0);
 analogWrite(11, 100);


 delay(2000);
`

The result of this test can be seen in the video below.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Gream7-C52o" title="ECE 4160: Lab 3 Three Sensors in Parallel" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 

While it may be hard to see, the difference in speeds can be heard in the video. 

### Short video of both wheels spinning (with battery driving the motor drivers)

### Picture of all the components secured in the car (Consider labeling your picture if you can’t see all the components)

### Lower limit PWM value discussion

### Calibration demonstration (discussion, video, code, pictures as needed)

### Open loop code and video

