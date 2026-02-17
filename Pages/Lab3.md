
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

**5. Check the polarity of these wires (positive on the battery should be connected to the positive terminal on your Artemis – this may mean that you need to connect opposite color wires).**
The polarity was actually oposite to how the board wanted it, so I soldered black to red and red to black. (refer to image)

**6. Power up your Artemis using only the battery and no connection through the USB C port. Try sending BLE messages back and forth from your laptop to ensure that the Artemis is powered on correctly and can send messages completely untethered.**
Insert video showing connection and data transfer.
   
## Using the Arduino library manager, install the SparkFun VL53L1X 4m laser distance sensor library.
Done

## Connect the QWIIC break-out board to the Artemis
Done

## Connect the first ToF sensor to the QWIIC breakout board.
**1. You will have to cut one end of a QWIIC cable and solder the other to your sensor. You have two long cables and two short ones, choose wisely.**
Done. Discuss choice for wire lengths, add image of setup next to the car

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
**3. The figure below is an example from 2020, when students measured the accuracy and repeatability in different lighting conditions, and timing for various code setups (these are not all required tasks for this year), however, we highly recommend generating your plots in the Jupyter notebook to gain more familiarity with the environment, e.g. using matplotlib.**


## Connect to Prelab
**Using notes from the pre-lab, hook up both ToF sensors simultaneously and demonstrate that both work.**
In order to use both ToF sensors at the same time, there are two options. 
1. We change one of the addresses of the ToF sensors so that they have different ones. To do this, use the Xshut pin on the sensor whose addres you're not changing (to turn it "off"), and then use the set.I2CAddress() function from the Sparkfun library to change the address (the default address of the Tof is 0x52/0x53 (for read/write), and the default addres of the I2C is 0x69, so I would just need to pick an address that is different than those. Then I can turn on the other ToF sensor, and it has the default address. I am not sure how this would affect the I2C address of the IMU device, but I think it will be okay)
2. We ignore the data from one of the ToF sensors whenever we want data from the other, by turning the one we don't want "off" (turning on the Xshut pin). This might be easier programming wise, but I like the other way better.

I chose the first method. It works well. I haven't tried it with the IMU as well yet.

## Speed it up
**In future labs, it is essential that the code executes quickly, therefore you cannot let your code hang while it waits for the sensor to finish a measurement. Write a piece of code that prints the Artemis clock to the Serial as fast as possible, continuously, and prints new ToF sensor data from both sensors only when available.**
**1. The distanceSensor.checkForDataReady() routine can be called to check when new data is available.**
**2. How fast does your loop execute, and what is the current limiting factor?**

## Edit Lab 1 Stuff
**Finally, edit your work from Lab 1, such that you can record time-stamped ToF data and IMU data for a set period of time, and then send it over Bluetooth to your computer.**

## Include a plot of the ToF data against time.

## Include a plot of the IMU data against time.

# Write-up

## Prelab
**1. Note the I2C sensor address**
**2. Briefly discuss the approach to using 2 ToF sensors**
**3. Briefly discuss placement of sensors on robot and scenarios where you will miss obstacles**
**4. Sketch of wiring diagram (with brief explanation if you want)**
## Lab Tasks
**1. Picture of your ToF sensor connected to your QWIIC breakout board**
**2. Screenshot of Artemis scanning for I2C device (and discussion on I2C address)
Discussion and pictures of sensor data with chosen mode**
**3. 2 ToF sensors and the IMU: Discussion and screenshot/video of sensors working in parallel**
**4. Tof sensor speed: Discussion on speed and limiting factor; include code snippet of how you do this**
**5. Time v Distance: Include graph of data sent over bluetooth (2 sensors)**
**6. Time v Angle: Include graph of data sent over bluetooth**
