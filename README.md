# Highway Driving: Path Planning Project

This project is the second projecty in the Self-Driving Car Engineer Nanodegree Program. 

[//]: # (Image References)
[startup]: ./imgs/startup-1.png
[lane_change]: ./imgs/lane_change-1.png
[final_image]: ./imgs/final-1.png

### Goals

In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

![][startup]

#### The map of the highway is in data/highway_map.txt

Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

### Simulator.

You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).  

To run the simulator on Mac/Linux, first make the binary file executable with the following command:

```shell
sudo chmod u+x {simulator_file_name}
```

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.

### Model Documentation

This project was completed using the directions provided in the *Project Q&A* section. As such we have utilized similar algorithms as Aaron & David viz. using the *spline* to derive smooth paths and using lane numbers to denote the left, middle and right lanes.

#### Compilation

The compliation was error free and didnt require any changes to the *cmake* files that were already provided in the repo. The only addition we had was adding the ```spline.h``` file in the src directory. This was obtained from the online repoistory: http://kluge.in-chemnitz.de/opensource/spline/,

#### Valid Trajectories

The simulation was run for a distance of 5 miles without any incidents. The entire time the car was able to obey the speed limit and keep within the maximmum allowed acceleration and jerk. The car was able to switch lanes when necessary: behind a slow moving car and noone in adjacent lanes. I also added additional logic to ensure that car's *rest* state was in the center lane: the car would move to the center lane after its done passing any cars and there are no cars in the center lane next to it.

![][final_image]

### Reflection

Most of the code required for running the project was already provided as part of the code repo. The real meat of the path planning algorithm can be found in [lines 102 to 285](./src/main.cpp#L102) of ```main.cpp```. This section deals with a few different key aspects

#### Understanding the Environment

This section deals with trying to ascertain the environment around the car using sensor fusion data. Is there a slow moving car ahead of us in our lane? If so, are there cars in our neighboring lanes hindering a lane change. This will be used to determine whether a lane change is necessary/feasible. For this purpose, we tag every other car as *dangerous* or not by ascertaining which lane they are in and the position it will be at the end of our car's planned trajectory. This section of the code can be found in [lines 116 - 153](./src/main.cpp#L116)

#### Lane Change Action

Now that we have the necessary information to decide whether to change lanes or not, the section in [lines 155 - 176](./src/main.cpp#L155) executes the action. Depending on the previous section, this could mean: execute a lane change, slow down or speed up (if it has switched lanes after a slow down). This differential is captured in the ```change_vel``` variable.

![][lane_change]

#### Trajectory planning

The rest of the code ([lines 177 - 285](./src/main.cpp#L177)) deals the with the generation of the trajectory for the car to follow. This obviously then takes information from the previous two sections to ensure a smooth and effective plan.

For this we used the same methodology suggested by the classroom and used a spline calculation. This utilizes two points from the previous trajectory together with three points in the distance (30, 60 and 90m from the position) to initiallize a spline calculation. As mentioned, to make the claculations less intensive the coordinates were transformed to local coordinates. In the intial condition scenario, the cars current position was used instead of the points from the previous trajectory.

The trajectory is further smoothened and made continuous, by copying the past trajectory points into the new trajectory generated. This is in addition to the last two points from the previous trajectory. As the trajectory list of points (50), is already almost filled out, only the remaining points need to be generated as part of the spline. 

These were created in accordance to the [project rubric](https://review.udacity.com/#!/rubrics/1971/view)
