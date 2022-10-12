# Robot Localization
**A Computational Introduction to Robots**

Allison Li

![Robot localization demo](./media/demo_2.gif)

## Overview
The goal of this project was to implement a [particle filter](https://en.wikipedia.org/wiki/Particle_filter) to localize a robot within a known map. 
## Implementation
I implemented the particle filter in 5 steps.
### 1. Generate Particle Cloud
Create a set of particles normally distributed about a given initial robot pose estimate.

### 2. Update Particle With Odometry (Movement Model)
Given the robot's current and previous odometry (position and orientation in the odometry frame), calculate a relative transform from the previous to the current odometry. Then, apply this transform to each particle with Gaussian noise. 

### 3. Update Particle With Laser (Sensor Model)
For each particle, calculate the positions of the lidar scans relative to the particle's pose. Then, calculate the distance from each scan to the nearest object in the map and take the average of this distance for all 360 scans. The closer the particle's pose matches with the robot's pose, the closer this average will be to zero. 
The particle's weight is set as the probability of the distance average given a normal distribution centered at 0. All of the particle weights are then normalized to sum to 1.

### 4. Update Robot Pose
Set the robot pose as the mean position (x-y coordinates) and orientation of the particle cloud. Fix the map to odometry frame transform using this new pose.

### 5. Resample Particles
Create a probability distribution using the particle weights and select a new set of particles from this distribution. 

## Design Decisions
The main design decision I debated during this project is how to determine a weight for the particles. Initially, I tried to make the weight directly inversely proportional to the average distance between laser scans and obstacles in the map (i.e. 1/distance), however, I ran into issues with how this scaled. A very small average would result in a huge weight and a large average would result in a very small weight, thus after the weights were normalized they would be too extreme. In the end, I chose to use a normal distribution and set the weights as the value of the distribution at the average obstacle distance. This allowed for a more gradual decrease in weight if the distance was relatively close to zero.

## Challenges
A challenge I ran into was code efficiency. My particle filter would update only every 30 seconds or so initially. I fixed this issue with converting loops to array or matrix operations. The occupancy field class functions can input and output arrays, so I took advantage of that to send in all of my scan points as an array rather than looping through them. The scipy.stats.norm function pdf() can operate on arrays as well, so I was able to calculate my weights all at once. I also used matrix operations to reduce the computational resources needed to convert all of the laser scans to cartesian coordinates in each particle's frame.

## Next Steps
Given more time I would like to solve the robot kidnapping problem, or at least begin to make my algorithm able to handle more inaccurate initial pose estimates. Currently, the particle cloud generates relatively tightly around the inital pose estimate, so a bad pose estimate can throw off the localization significantly.

## Takeaways
A big takeaway is to take as much advantage of array and matrix operations as possible, as they are much more computationally efficient for Python than loops. Matrix transformations are very powerful to make code more generalized and for overall efficiency, so I will keep that in mind for future projects.
