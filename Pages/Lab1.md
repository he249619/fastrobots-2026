--- 
title: "Lab 1"
description: "Lab Description and Report"
layout: default
---

# Lab 1: The Artemis Board and Bluetooth

**Add titles for all tasks**

## Lab 1A

### Pelab

For this all following labs, being able to program the Artemis board and communicate with it with our personal lapton via Bluetooth is vital. To do this, I updated my Arduino IDE and Sparfun Appollo3 boards manager.

### Task 1

I connected the Artemis board to my computer using the USB C output on my laptop to connect to the USB C input on the board.

**Insert images of the cord, the computer port, and the board port**

### Task 2

To begin programming the board, I burned Blink, from Arduino's example programs, onto the Artemis board. This caused the blue LED on the board to turn on for one seconds and off for one second, repeatedly.

<!-- This code to display the video was adapted from Chat GPT -->

<iframe width="560" height="315" src="https://www.youtube.com/embed/DG_8jBnNdu4"
  title="ECE 4160: Lab 1A Blink" frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>


### Task 3

**I forgot what this program does, redo and upload any necessary material**

# <img src="Images/Lab 1/lab1A_task3.png" alt="Task 3!" style="width: 610px; height: 397px"/>

### Task 4

To get familiar with the capabilities of the Artemis board to handle analogue data, I uploaded the Apollo3's Example2_analogRead code to the board. This program causes the board to read in data from its on-board temperature sensor. The original code prints this raw analog value, along with data about internal voltages and timing on the board. 

**Add image from the output, and then change the code to output correct image and maybe include this**

**Also include video of temperature changing**

# <img src="Images/Lab 1/lab1a_task4.png" alt="Task 3!" style="width: 671px; height: 295px"/>


### Task 5

Finally, I uploaded the Example1_MicrophoneOutput from the RedBoard Artemis Nano examples in order to test out the pulse density microphone (PDM) on the Artemis board. By trying to move my voice to different pitches **I think, I need to double check**, I was able to change the highest recorded frequency that the PDM detected and the program output to the serial monitor.

**Insert video of the singing and changing frequency**

## Lab 1B

### Pelab

### Configuration

In order to set up proper Bluetooth communication between my lapton and the board, I had to determine the MAC address of the board. as well as the UUID associated with my board, both of which are 

# <img src="Images/Lab 1/lab1b_artemis_MAC_address.png" alt="Task 3!" style="width: 352px; height: 30px"/>

Additionally, I needed to determine the UUID associated with the board, which  was done through my laptop using Jypeter Notebook.

# <img src="Images/Lab 1/lab1b_more_ble_stuff.png" alt="Task 3!" style="width: 373px; height: 91px"/>

### Task 1



## Discussion
