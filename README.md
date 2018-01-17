# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 50 m/s^3.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```


## Project walkthrough

### 1. Get the car moving:
  The car will move following the waypoints we receive from the data, so, in order to make the car start moving and going through the waypoints, we have to set the acceleration and the path the car will follow. The path is made by the 50 waypoints in front of the car and the acceleration is limited to 0.224 miles/s^2. That will be enought to get the car moving, but it will keep accelerating forever and speeding up the speed limit, and we do not want that. The car will also go through the forest when the lane is not straight (Lines 401 - 422)
  
### 2. Keep the car on the lane:
  For this purpose we learned during previous lessons about Frenet coordianates. Frenet coordinates are a way of representing position on a road using the variables s and d. The s coordinate represents distance along the road (also known as longitudinal displacement) and the d coordinate represents side-to-side position on the road (also known as lateral displacement).
  
  ![frenet 1](./img/frenet-1.png) ![frenet 2](./img/frenet-5.png)
  
  So we just have to get the Frenet coordinates and follow a new "straight" path.
  
### 3. Do not be faster than the speed limit:
  Since we do not want our car to have fees, we have to set a `MAX_SPEED` variable that the car will have to respect. If the speed is higher than the limit then it has to stop accelerating.
  
### 4. Drive smooth please:
  No body wants a car that is accelerating and braking super hard, neither going to right or left lane breaking our necks so, in order to make smooth trajectories we have two variables that will help us: 
  * `MAX_ACC` will say how much the car can accelerate and decelerate in miles/s^2.
  * `spline` this is not a variable but a library that provides a function that makes smooth trajectories when you have a list of points.
  
  
### 5. Do NOT crash my car:
  Here is were the magic starts, since we want to take our car and passengers without damage, we need to know what the other cars in the road are doing. For this purpose we have the sensor fusion data. This data tells us where the other cars are (in x,y coordinates and s,d Frenet coordinates) and the speed they are going. This data is enought to avoid collisions and danger situations. What I have done here is to check the nearest lanes cars (speed and position related to us) and select the best option (keep my lane or change lane).
  (Lines 279 - 297)


### 6. Smooth and safety and... FASTER:
  We can not go allways behind a slow car since our goal is to reach our destination as fast, with the speed limit on mind, as possible. Because of that if we are behind a slow car and we can not overtake because there are other vehicles in the close lanes, the car will get the speed of the car ahead (Line 286) and copy it (Line 311), in order to be allways as fast as possible.
