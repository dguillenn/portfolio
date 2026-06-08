# Marker Visual Localization

**Skills:** Mobile Robotics, Computer Vision, Visual Localization, AprilTags, Pose Estimation, PnP, Coordinate Transformations, Sensor Fusion, Python

## Overview

This project implements a visual localization system for a mobile robot using AprilTag fiducial markers.

The robot estimates its global pose by detecting AprilTags with an onboard camera, computing the relative camera-to-tag transformation using Perspective-n-Point (PnP), and transforming this information into world coordinates using the known locations of the markers.

To ensure continuous localization, visual measurements are combined with odometry. When markers are temporarily unavailable, the robot propagates its pose estimate using odometric motion until visual localization becomes available again.

## Objective

Develop a localization system capable of:

- Detecting AprilTags from camera images.
- Estimating camera pose using Perspective-n-Point (PnP).
- Transforming local observations into world coordinates.
- Correcting odometry drift through visual measurements.
- Maintaining localization during temporary visual loss.
- Filtering noisy visual estimates to improve stability.

## Technologies

- Python
- OpenCV
- AprilTags
- Perspective-n-Point (PnP)
- Homogeneous Transformations
- Visual Localization
- Mobile Robotics

## System Architecture

The localization pipeline follows a vision-first approach with odometry fallback.

Whenever a valid marker is detected:

1. The AprilTag is identified in the camera image.
2. The relative pose between the camera and the tag is estimated using PnP.
3. Coordinate frame corrections are applied.
4. The pose is transformed into world coordinates.
5. The resulting robot pose is validated and filtered.

If no valid visual information is available, the robot continues estimating its pose using odometry increments.

This approach combines the global consistency of vision with the continuity provided by odometry.

## AprilTag Detection

Each control cycle:

1. An image is captured from the onboard camera.
2. The image is converted to grayscale.
3. The AprilTag detector searches for visible markers.
4. The detector returns:
   - Tag ID
   - Pixel coordinates of the four corners

The detected marker borders are displayed in the graphical interface to verify detection quality and marker orientation.

## Pose Estimation Using PnP

The relative pose between the camera and the detected marker is computed using OpenCV's Perspective-n-Point algorithm.

Each AprilTag is modeled as a square planar object with known dimensions:

```text
0.24 m × 0.24 m
```

The four 3D corner coordinates defined in the marker reference frame are matched with their corresponding image projections.

Using the camera intrinsic matrix, OpenCV estimates:

- Rotation vector (`rvec`)
- Translation vector (`tvec`)

The implementation uses:

```text
SOLVEPNP_IPPE_SQUARE
```

which is specifically designed for planar square markers and provides stable pose estimates for AprilTags.

## Homogeneous Transformations

The rotation vector returned by PnP is converted into a rotation matrix using Rodrigues' formula.

Rotation and translation are then combined into a homogeneous transformation matrix:

```text
Camera → Tag
```

To obtain the camera pose relative to the marker, the transformation is inverted:

```text
Tag → Camera
```

Homogeneous transformations simplify the chaining of rotations and translations across multiple coordinate systems.

## Coordinate Frame Alignment

The coordinate frame used by OpenCV differs from the robot reference frame.

### OpenCV Camera Frame

```text
X → Right
Y → Down
Z → Forward
```

### Robot Frame

```text
X → Forward
Y → Left
Z → Up
```

To align both systems correctly, two fixed rotations are applied:

- Rotation around the X-axis.
- Rotation around the Z-axis.

The transformations are applied in a specific order using matrix multiplication.

Without this correction, estimated positions and orientations would not match the robot's actual motion.

## World Coordinate Transformation

The pose of every AprilTag is known beforehand and stored in a configuration file.

For every detected marker:

1. A world-to-tag transformation is built using the marker's known position.
2. The tag-to-camera transformation obtained from PnP is applied.
3. The resulting transformation provides:

```text
World → Camera
```

From this matrix:

- Robot position is extracted from the translation component.
- Robot yaw is extracted from the rotation component.

This yields a complete global pose estimate:

```text
(x, y, yaw)
```

## Visual Pose Selection

Multiple markers may be visible simultaneously.

To improve accuracy, the system selects the marker with the smallest camera distance because closer observations generally provide more reliable pose estimates.

This strategy reduces uncertainty and improves localization quality.

## Noise Filtering

Visual localization becomes increasingly noisy as the distance to the marker grows.

To prevent unstable pose estimates, a jump rejection mechanism was implemented.

### Validation Strategy

A maximum allowed position change is defined.

If:

- The detected marker is far away, and
- The estimated pose differs excessively from the previous estimate,

the measurement is rejected.

This prevents unrealistic jumps while still allowing genuine corrections when the robot moves.

## Odometry-Based Localization

When visual measurements are unavailable, localization continues using odometry.

The system stores:

- The last valid visual pose.
- The odometry pose associated with that visual update.

During visual loss:

1. Current odometry is compared with the stored odometry pose.
2. Relative motion is computed.
3. The motion increment is applied to the last visual pose.

This maintains a continuous pose estimate until visual localization becomes available again.

## Robot Motion and Safety

To evaluate localization performance during motion, the robot moves continuously through the environment.

A simple obstacle avoidance behaviour was implemented using laser measurements.

### Safety Behaviour

When an obstacle is detected:

1. Forward motion stops.
2. The robot rotates in place.
3. Motion resumes once the path is clear.

This allows long-duration localization tests without collisions.

## Challenges and Solutions

### Noisy Localization at Long Distances

#### Problem

Markers observed from large distances produced unstable pose estimates.

#### Solution

Distance-based filtering and jump rejection were introduced to discard unreliable measurements.

### Pose Discontinuities

#### Problem

Large jumps occasionally appeared when switching from odometry-based localization back to visual localization.

#### Solution

New visual estimates are compared against the previous valid pose and rejected if the difference exceeds predefined thresholds.

### Coordinate Frame Inconsistencies

#### Problem

The coordinate systems used by OpenCV and the robot platform were not aligned.

#### Solution

Fixed rotational transformations were applied to correctly express camera poses in the robot reference frame.

## Results

The system successfully estimates the global pose of the robot using AprilTag observations while maintaining localization through odometry during temporary visual loss.

The combination of visual corrections, coordinate frame alignment, and measurement filtering significantly reduces odometry drift while preserving localization stability.

## Demonstration

### Video

[Video](https://youtu.be/6yYFogPnERw)

## Conclusions

This project demonstrates a complete marker-based visual localization pipeline for mobile robots.

By combining AprilTag detection, PnP pose estimation, homogeneous transformations, coordinate frame alignment, and odometry propagation, the system achieves robust and continuous localization in environments instrumented with visual markers.

The results highlight the effectiveness of vision-based localization for correcting odometry drift while maintaining reliable pose estimates during temporary loss of visual information.
