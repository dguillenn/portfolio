# Laser Mapping

**Skills:** Mobile Robotics, SLAM Fundamentals, Occupancy Grid Mapping, Frontier Exploration, BFS, LIDAR Processing, Autonomous Navigation, Python

## Overview

This project implements an autonomous exploration and mapping system capable of navigating an unknown environment while generating an occupancy grid map using LIDAR data.

The robot continuously explores unexplored regions, updates a probabilistic occupancy map, and replans its trajectory as new information becomes available.

The exploration strategy is based on frontier detection, while the map is built using a probabilistic log-odds occupancy model.

## Objective

Develop an autonomous exploration system capable of:

- Detecting obstacles and free space using LIDAR data.
- Building an occupancy grid map of an unknown environment.
- Identifying frontiers between explored and unexplored regions.
- Navigating safely toward unexplored areas.
- Replanning dynamically as the map evolves.

## Technologies

- Python
- LIDAR Sensors
- Occupancy Grid Mapping
- Frontier-Based Exploration
- Breadth-First Search (BFS)
- Probabilistic Robotics

## System Overview

The robot explores the environment by repeatedly searching for the closest frontier and navigating toward it.

A frontier is defined as a free cell that is adjacent to at least one unknown cell.

The process is repeated until no reachable frontiers remain, meaning the entire environment has been explored.

The robot maintains its position in map coordinates using the provided simulator pose and the `poseToMap()` conversion function.

This conversion allows transforming real-world coordinates into occupancy grid coordinates and is also used to compute the robot heading direction inside the map.

## Frontier Search

### BFS Frontier Detection

At the beginning of every exploration cycle, the robot performs a Breadth-First Search (BFS) to locate the nearest frontier.

To improve performance, the search does not expand pixel by pixel.

Instead, neighboring nodes are generated every 15 pixels.

This significantly reduces the number of visited nodes while maintaining smooth exploration behaviour.

The BFS expands from the robot position and evaluates cells that satisfy two conditions:

- The cell is free.
- The cell is safe.

As soon as a frontier is found, the shortest path is reconstructed and converted into a list of waypoints.

The robot then follows these waypoints sequentially.

### Safety Region

A safety verification function was implemented to avoid selecting waypoints too close to obstacles.

For every candidate pixel, a rectangular safety region is evaluated.

```text
25 × 25 pixels
```

This corresponds to a clearance of:

```text
12 pixels in every direction
```

The dimensions were experimentally determined by observing the size of walls and obstacles in the occupancy map.

This prevents the robot from selecting waypoints that are technically free but unsafe in practice.

### BFS Failure Recovery

Occasionally, the robot may stop too close to a wall.

In those situations, the current robot position is considered unsafe and BFS cannot expand.

To solve this problem, a recovery behaviour was added:

- The robot slowly moves backward.
- BFS is executed again.
- Exploration continues once a safe starting position is available.

## Probabilistic Occupancy Mapping

### Log-Odds Representation

The map is built using a probabilistic occupancy grid based on log-odds updates.

For every laser beam:

1. The beam angle is computed using the robot yaw.
2. The measured distance is obtained.
3. The endpoint is projected into the map.

During testing, the effective maximum laser range was observed to be approximately:

```text
3.5 meters
```

Therefore:

- If an obstacle is detected, the endpoint is marked as occupied.
- If no obstacle is detected, the beam is assumed to travel the full 3.5 meters.

### Free Space Update

All cells traversed by the laser beam are marked as free.

For each traversed cell:

```text
log_odds += log_free
```

### Occupied Cell Update

If the laser beam ends on an obstacle:

```text
log_odds += log_occ
```

is applied to the endpoint.

Over time, repeated observations cause the occupancy probabilities to converge.

Interpretation of log-odds values:

```text
Positive value → Occupied
Negative value → Free
Near zero      → Unknown
```

This produces a continuously improving occupancy map.

## Path Following

Once a valid frontier path has been generated, the robot follows the corresponding waypoints.

### Angular Controller

A proportional controller is used for angular velocity:

```text
w = K · angular_error
```

The angular error is computed in map coordinates.

To estimate the robot heading:

1. The robot position is converted to map coordinates.
2. A point located one meter ahead of the robot is converted as well.
3. The heading angle is computed from these two points.

The resulting heading is compared with the direction toward the current waypoint.

### Navigation Behaviour

The robot:

- Rotates toward the waypoint.
- Moves forward only when the heading error is sufficiently small.
- Advances to the next waypoint when it gets close enough.
- Stops when the current frontier is reached.

This produces smooth and reliable navigation.

## Dynamic Safety Checks

Even after a path has been generated, waypoint safety is continuously re-evaluated.

Since the occupancy map changes over time, a waypoint that was originally safe may later become unsafe.

When this happens:

1. The current path is discarded.
2. A new BFS search is launched.
3. The robot backs away if necessary.
4. Exploration resumes using the updated map.

This makes the system robust in cluttered and dynamic environments.

## Challenges and Solutions

### Robot Stuck Near Walls

#### Problem

Sometimes BFS could not expand because the robot was located too close to a wall.

#### Solution

A recovery strategy was implemented that applies a small backward velocity until a safe starting position is available.

### Maximum Laser Range

#### Problem

The simulator did not explicitly return a maximum range measurement.

#### Solution

Experimental testing showed that the effective maximum range was approximately 3.5 meters.

All no-return measurements were therefore treated as 3.5-meter readings.

### Exploration Performance

#### Problem

A pixel-by-pixel BFS was computationally expensive.

#### Solution

The search was modified to advance in 15-pixel increments.

This significantly reduced computational cost while maintaining exploration quality.

### Collisions with Obstacles

#### Problem

Initially, BFS only checked whether a pixel itself was free.

This caused the robot to navigate dangerously close to walls and shelves.

#### Solution

A safety-region validation function was implemented to verify both the candidate cell and its surroundings.

This introduced the required clearance for safe navigation.

### Laser Mapping Errors

#### Problem

Some laser updates incorrectly modified cells located behind obstacles.

#### Solution

Laser rays were truncated whenever an obstacle was detected.

This prevented updates from affecting cells beyond the obstacle.

## Results

The robot successfully explored the entire environment while simultaneously building an occupancy map.

The combination of frontier-based exploration, probabilistic mapping, and continuous path validation produced reliable exploration behaviour even in complex warehouse-like environments.

### Demonstration Videos

#### Perfect Localization

[Video](https://youtu.be/kQ0iiaYNFyQ)

#### Odometry Localization 1

[Video](https://youtu.be/bWlhDlHkJS4)

#### Odometry Localization 2

[Video](https://youtu.be/E-N3ybarGxA)

#### Odometry Localization 3

[Video](https://youtu.be/IGq5mws1vDU)

## Localization Comparison

### Perfect Localization

Using the simulator ground-truth position, the robot generates an accurate occupancy map and successfully explores the entire environment.

### Odometry Localization

When relying exclusively on odometry, localization errors accumulate over time.

#### Odom1

The robot begins with a correct estimate but small localization errors gradually appear.

#### Odom2

The accumulated error causes visible map deformation and reduced mapping accuracy.

#### Odom3

Localization drift becomes significant enough for the robot to collide with an obstacle, demonstrating the limitations of pure odometry-based navigation.

## Conclusions

This project demonstrates a complete autonomous exploration pipeline combining frontier detection, occupancy grid mapping, path planning, and reactive safety mechanisms.

The probabilistic log-odds representation provides robust map construction, while frontier-based exploration enables efficient coverage of unknown environments.

The comparison between perfect localization and odometry-based localization also highlights the importance of accurate localization systems for reliable autonomous mapping.
