title: "Lab 10"
description: "Lab Description and Report"
layout: default
---

# Lab 10: Grid Localization using Bayes Filter


# <img src="Images/Lab 9/world_pic.jpg" style="max-width:90%"/>


```cpp
```

<iframe width="560" height="315" src="https://www.youtube.com/embed/3K-tkRqZhZw" title="ECE 4160: Lab 9 Mapping" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen> </iframe> 


### Pre-lab: Using the Simulator

####  Open Loop Controller
The velocity command runs for 1 second, but that is because I force the system to maintain a specific velocity for 1 second by using `await asyncio.sleep(1)`. If I had not put this here, I would assume that the velocity command could be set once and stay on at the same velocity until it was changed to a different velocity value via a new velocity command.

It appears that the robot creates about the same shape every complete cycle, but it is not a perfect square. This can easily be deduced when looking at the plotted true position of the robot, which looks like a lot of square shaped objects rotated about their center, stacked upon one another. However, it does appear that the shape the robot traces out seems to rotate the same amount every cycle, leading me to believe that the robot is executing roughly the same shape.

```python
# Open Loop Control
while cmdr.sim_is_running() and cmdr.plotter_is_running():
    
    cmdr.set_vel(0.5, 0)
    await asyncio.sleep(1)
    cmdr.set_vel(0, 1.5708)
    await asyncio.sleep(1)

    pose, gt_pose = cmdr.get_pose()

    cmdr.plot_odom(pose[0], pose[1])
    cmdr.plot_gt(gt_pose[0], gt_pose[1])

```
####  Closed Loop Controller
I created a simple closed loop controller in order to perform basic obstacle avoidance on the robot. 

```python
minimum_distance_threshold = 0.5

# Closed Loop Control
while cmdr.sim_is_running() and cmdr.plotter_is_running():

    sensor_values = cmdr.get_sensor()

    while(sensor_values <= minimum_distance_threshold):
        cmdr.set_vel(0, 1.5708) # turn 10 degrees to the left
        await asyncio.sleep(0.2)
        sensor_values = cmdr.get_sensor()

    # if (sensor_values <= minimum_distance_threshold):
    #     cmdr.set_vel(0, 0.523599) # turn 30 degrees to the left
    #     await asyncio.sleep(1)
    
    
    cmdr.set_vel(2, 0)

    pose, gt_pose = cmdr.get_pose()

    cmdr.plot_odom(pose[0], pose[1])
    cmdr.plot_gt(gt_pose[0], gt_pose[1])

```


Whenever the robot was within 0.5 meters of an obstacle directly in front of it, as measured by the distance center on the front of its chassis, the robot car would turn 90° and continue turning until the sensor reported a distance greater than 0.5 meters. Since all of the obstacles within the robot’s world had right corners, I figured that a 90° rotation would be sufficient in allowing the robot to avoid most obstacles. In addition to this, I made sure that the robot moved slowly so that it could rotate in time before hitting an obstacle in front of it. I originally implemented it so that the robot moves at about 0.5 m/s, but I also noticed that the obstacle avoidance seemed to work just as fine when the robot moved at 2 m/s. I do not believe this result would be as easily replicable in the real world because it is possible the simulator doesn’t account for the momentum that the car has when moving quickly.

I noticed that the robot will run into an obstacle if it is not directly in its line of sight, but the obstacle still exists within the robot’s path. For example, this seems to happen when the robot’s sensor’s path of measurement barely misses a close wall or corner. The robot then believes that its closest obstacle is actually much farther away than it truly is, causing it to run into a wall and change its course.

One possible solution to this could be to have the robot move forwards until it is a certain distance from the obstacle in front of it, and then once it has surpassed that threshold the robot could rotate a full 360° and incrementally take distance measurements. The robot could then go between whichever two angles resulted in consecutive measurements that had the largest average distance. Using the heading between these angles would hopefully avoid the situation of the robot unexpectedly running into walls.


### Lab

Perform Grid localization for the sample trajectory.
You may modify the trajectory for better results.
Attach a video of the best localization results (along the entire trajectory).


In each iteration of the bayes filter, you will need to go through all possible previous and current states in the grid to estimate the belief. Think about how many loops would be required to perform this.
Given that your grid has 12×9×18 = 1944 possible states, there is lot of computation involved in each iteration of the bayes filter. Hence, you need to write efficient code, especially in Python, if you want your entire estimation process to run within a couple of minutes. Try to get the running time of the prediction step and the update step functions to be within a couple of seconds; shorter running times may prove beneficial for testing and debugging. Here are some ways to reduce processing time:
If the probability of a state (i.e grid cell) is 0, then we can skip that grid cell in the inner loops of the prediction step of the bayes filter. (i.e. only in multiplicative terms, since multiplying any value with a 0 results in a 0). In fact, if a state has a probability less than 0.0001, we can skip those states as they don’t contribute a lot to the belief and thus can reduce the computation time. Note that since you are skipping some states, the sum of the probabilities across the grid may no longer sum to 1. So you will need to make sure you normalize at the end of your prediction step and update step (which you already do as per the algorithm).
Reduce unnecessary interim variables. This can lead to slow processing times, especially in Python.
Use Numpy for faster matrix operations instead of element wise operations. Numpy is faster if you can use matrix-like operations because the backend processing happens in C.
HINT: The gaussian function of class BaseLocalization can handle Numpy variables; think about how to perform Numpy operations in the update step. You may not even need the sensor_model function if you perform grid indexing to directly compute the sensor noise model in the update_step function.
To demonstrate that you’ve successfully completed the lab, please upload a brief lab report (<600 words), with code (not included in the word count), photos, and videos documenting that everything worked and what you did to make it happen. Include the most probable state after each iteration of the bayes filter along with its probability and compare it with the ground truth pose. Write down your inference of when it works and when it doesn’t.
Please use screen-recording tools to record any necessary media.

