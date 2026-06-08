# Global Navigation

**Skills:** Mobile Robotics, Autonomous Navigation, Path Planning, Gradient Path Planning (GPP), BFS, Occupancy Grids, Python

## Overview

This project implements a Gradient Path Planning (GPP) system for autonomous global navigation.

Given a user-selected destination, the robot computes a safe path through the environment, avoiding obstacles while maintaining smooth and efficient motion. The navigation strategy combines an attractive scalar field that guides the robot toward the goal with a repulsive penalty field that keeps it away from obstacles.

## Objective

Develop a global navigation system capable of:

- Computing a safe path to a target location.
- Avoiding obstacles while navigating.
- Following the generated path automatically.
- Reaching the destination smoothly and efficiently.

## Technologies

- Python
- Occupancy Grid Maps
- Breadth-First Search (BFS)
- Gradient Path Planning (GPP)
- Autonomous Navigation

## Environment Representation

The environment is represented using an occupancy grid map.

Grid values are defined as:

```text
255 → Free space
0   → Obstacle
```

Since the robot operates using real-world coordinates `(x, y)`, a coordinate transformation function was implemented to convert grid coordinates into world coordinates.

This allows the robot to accurately relate its position to the map.

## Attractive Scalar Field

The main navigation mechanism is based on an attractive scalar field.

Each free cell in the map receives a value representing its distance to the goal.

The field is generated using a Breadth-First Search (BFS):

1. The search starts from the goal cell.
2. Expansion is performed in all directions, including diagonals.
3. Straight movements have a cost of `1`.
4. Diagonal movements have a cost of `√2`.
5. The search stops early when the robot position is reached.

Cells closer to the destination receive lower values.

As a result, the robot can navigate simply by moving toward neighboring cells with smaller values.

## Penalty Field

The attractive field alone does not prevent the robot from moving too close to obstacles.

To improve safety, a repulsive penalty field is added.

For every visited cell:

1. A local neighborhood around the cell is inspected.
2. Nearby obstacles are detected.
3. A penalty proportional to the inverse of the distance is applied.

The closer the obstacle, the larger the penalty.

The final navigation field is obtained by combining both fields:

```text
total_field = scalar_field + penalty_field
```

This allows the robot to follow efficient paths while maintaining a safe distance from walls and obstacles.

## Navigation Strategy

### Next Cell Selection

To determine the next movement direction, the robot evaluates nearby cells around its current position.

Initially, all cells within a square neighborhood were evaluated, but this proved computationally expensive.

To improve performance, only the outer ring of the neighborhood is evaluated.

The robot:

1. Examines all valid cells on the outer ring.
2. Selects the cell with the lowest field value.
3. Uses that cell as the next navigation target.

This significantly reduces computational cost while maintaining path quality.

### Angle Control

The robot computes the desired heading toward the selected cell.

The angular error is calculated as:

```python
angle_error = desired_angle - car_pose.yaw
```

To ensure the robot always rotates using the shortest direction, the angle is normalized:

```python
if angle_error > pi:
    angle_error -= 2 * pi

if angle_error < -pi:
    angle_error += 2 * pi
```

This keeps the error within:

```text
[-π, π]
```

### Motion Control

The robot then:

- Rotates according to the angular error.
- Moves forward toward the selected cell.
- Reduces speed when approaching the goal.
- Reduces speed during sharp turns.
- Stops when the destination is reached.

## Challenges and Solutions

### High Computational Cost

#### Problem

The first implementation evaluated every cell inside a square neighborhood around the robot.

Although functional, the approach became computationally expensive and slowed down navigation.

#### Solution

The algorithm was optimized by:

- Replacing nested loops with NumPy vectorized operations.
- Evaluating only the outer ring instead of the full neighborhood.

This significantly reduced the number of processed cells and improved overall performance.

## Results

The implemented Gradient Path Planning algorithm successfully generates safe and efficient paths through the environment.

The combination of attractive and repulsive fields allows the robot to navigate toward the goal while maintaining safe distances from obstacles.

### Demonstration Video

[Video](https://rr2---sn-vg5obxn25po-h5qee.googlevideo.com/videoplayback?expire=1780967963&ei=m_kmat3TNOuO9fgP8om2gQ0&ip=92.185.20.102&id=b6e19e29a8972b82&itag=22&source=blogger&requiressl=yes&xpc=Egho7Zf3LnoBAQ==&cps=188&met=1780939163,&mh=gP&mm=31&mn=sn-vg5obxn25po-h5qee&ms=au&mv=m&mvi=2&pl=20&rms=au,au&susc=bl&svpuc=1&eaua=TFqxoVKvHBA&mime=video/mp4&vprv=1&rqh=1&dur=92.136&lmt=1763551034709509&mt=1780938827&txp=1311224&sparams=expire,ei,ip,id,itag,source,requiressl,xpc,susc,svpuc,eaua,mime,vprv,rqh,dur,lmt&sig=AHEqNM4wRQIhAOyrLIdFAR5Hg84eKR9XT7roSOcEUc2l9OFnokM4OdgZAiBEr5-3XZmEAIIdDcsOImdvXZIXsciNt5D1iLvhN22gEQ==&lsparams=cps,met,mh,mm,mn,ms,mv,mvi,pl,rms&lsig=APaTxxMwRQIgWYzPC1j-GEboXJJu3OGdkFSLj9J42qz2OloLv-97_5MCIQCmnYktNgT7CZH1HNQrtXclR3RwxKQsURKLMTCOsLz0-Q==&cpn=XoDrepIMsCldJaCc&c=WEB_EMBEDDED_PLAYER&cver=1.20260607.05.00-canary_experiment_1.20260603.06.00)

## Conclusions

This project demonstrates how Gradient Path Planning can be used to achieve autonomous global navigation using occupancy grid maps.

The attractive scalar field provides an efficient path toward the goal, while the penalty field improves safety by discouraging paths that pass close to obstacles.

The optimization of neighborhood evaluation and the use of vectorized operations greatly improved computational efficiency, making the system suitable for real-time navigation.
