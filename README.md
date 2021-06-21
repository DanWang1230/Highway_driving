# **Highway Driving**

The goal is to design a path planner that is able to create smooth, safe paths for the car to follow along a 3 lane highway with traffic. A successful path planner will be able to keep inside its lane, avoid hitting other cars, and pass slower moving traffic all by using localization, sensor fusion, and map data.

[//]: # (Image References)

[image1]: Result_image.png

---
### Code model for generating paths

The Project Q&A is really helpful. Based on that lecture, I created the path planner. And the planner works and generates no incidients for more than 4.32 miles, as shown in the following image. A video showing the performance of the path planner is uploaded [here](https://youtu.be/1Nu3SbrVvoc).

![alt text][image1]

Next I will talk in detail about the key points in building the planner.

#### 1. Reducing jerk and keeping acceleration low

This is achieved by applying the incremental change to the velocity.

```C++
if(too_close)
{
    ref_vel -= .224; //5m/s^2 under the acceleration limit 10m/s^2
}
else if(ref_vel < 49.5)
{
    ref_vel += .224;
}
```

#### 2. Generating smooth trajectories

Spline fitting tool is used here. Points (`ptsx`, `ptsy`) are set to the spline.

```C++
#include "spline.h"
```
Also, points from previous path (`previous_path_x`, `previous_path_y`) are included in the newly generated way points.

#### 3. Getting close to the car in front

Boolean `too_close` is the indicator for this incident. Here, the s value of my car is compared with the s value of the car in front.

```C++
if((check_car_s > car_s)&&( check_car_s - car_s < 30))
{
    too_close = true;
}
```
#### 4. Lane change left

Boolean `lane_change_left` is the indicator for this incident. Its default value is true. First, ensure the lane number is larger than 0 since cars in Lane 0 cannot make a left lane change. When the cars in the left lane are too close (in [-10， 20]), `lane_change_left` is set to false.

```C++
// check if lane change left is possible
if (lane > 0)
{
    // car in left lane
    if((d < 2+4*(lane-1)+2)&&(d > 2+4*(lane-1)-2))
    {
    if(check_car_s - car_s < 20 && check_car_s - car_s > -10)
    {
        lane_change_left = false;
    }
    }
} else
{
    lane_change_left = false;
}
```

#### 5. Lane change right

Boolean `lane_change_right` is the indicator for this incident. Its default value is true. First, ensure the lane number is less than 2 since cars in Lane 2 cannot make a right lane change. When the cars in the right lane are too close (in [-10， 20]), `lane_change_right` is set to false.

```C++
// check if lane change right is possible
if (lane < 2)
{
    // car in right lane
    if((d < 2+4*(lane+1)+2)&&(d > 2+4*(lane+1)-2))
    {
    if(check_car_s - car_s < 20 && check_car_s - car_s > -10)
    {
        lane_change_right = false;
    }
    }       
} else 
{
    lane_change_right = false;
}           
```


