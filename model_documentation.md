## Writeup
---
**Highway Driving**
---
References:
Udacity starter code provided in the tutorials and the Q&A  was used throughout.

### Initial Driving set up

The car was initially set to the middle lane (lane 1) with a velocity of 0 mph.

### Acceleration

To decide whether to speed up or slow down, the car looks at the position of all other vehicles on the road. If any of these vehicles are within the current lane of the vehicle and within 30m in front of the car, then the car will decelerate. If no cars are within 30m ahead of the car the car will accelerate by increasing the speed by 0.224 every time step until a maximum speed of 49.5mph is acheived, which prevents the car from going over the speed limit. 

### Lane following and trajectories

To allow for smooth lane following and trajectories the spline library is used. https://kluge.in-chemnitz.de/opensource/spline/spline.h This package allows us to easily interpolate a smooth line between points. 

The spline is applied to 5 key anchor points. The previous position of the car, the current position of the car and positions 30m, 60m and 90m along the road in the desired lane. These locations are then converted into the car coordinates to make the coordinates easier to understand.

The spline is then used to create 50 points appropriately spaced to provide the desired speed. Once this has been run once, the algorithm retains the points from the previous trajectory that weren't used and only adds new points required to make there be 50 points. This avoids any sudden changes in trajectory.

### Finite state machine.

A simple finite state machine is then used to decide whether to change lanes. If the car approaches a car in front and has to slow down it will consider 3 possible states.

1. change lane left
2. stay in lane
3. change lane right

Each of these scenarios is assessed and a cost applied. The cost function is relatively simple. 

1. change lane left
	 - No lane left - cost 1000
     - Lane left has a car within +- 30 m of the car - cost 1000

2. stay in lane
	- slow down for car in front - cost 300
    
3. change lane right
	 - No lane right - cost 1000
     - Lane right has a car within +- 30 m of the car - cost 1000
     
    
The below code is used to pick the lowest cost action.
```python 
	          // Pick the action with the lowest cost.
                    // move left
                    if ((cost_1 < cost_2) && (lane > 0)) 
                		{
                        std::cout << "LANE CHANGE minus"<< std::endl ;
                	 	lane = lane - 1;
                		}
                	// move right
               		if ((cost_3 < cost_2) && (cost_3< cost_1) && (lane <2))
                        {
                          std::cout << "LANE CHANGE plus"<< std::endl ;
                          lane = lane + 1;
                        }
          
```


This means that the car will stay in the current lane in the case where there isn't an adjacent lane that doesn't have a car just in front or just behind. There is no point changing lane if there is another car just in front. 

