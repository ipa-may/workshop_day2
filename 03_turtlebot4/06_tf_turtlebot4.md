# TF on TurtleBot4

## What is a TF ?

TF is the ROS 2 transform system.

It describes the spatial relationship between coordinate frames over time.

Examples:

- where `base_link` is relative to `odom`
- where the lidar frame is relative to the robot body
- where the camera frame is relative to the robot body

Without TF, ROS 2 nodes can receive sensor data, but they do not know where that data is located in space.

## Why TF matters on TurtleBot4

TF is used for:

- RViz visualization
- robot localization and navigation
- placing lidar scans in the correct position
- placing camera data in the correct position
- relating robot motion to a frame such as `odom`

## Main TF topics

On the TurtleBot4, the transform tree is published on:

```sh
/tf
/tf_static
```

Message type:

```sh
tf2_msgs/msg/TFMessage
```

Difference:

- `/tf` contains dynamic transforms that change over time
- `/tf_static` contains static transforms that do not change

## Important TurtleBot4 frames

From the live TurtleBot4 TF tree, common frames include:

- `odom`
- `base_link`
- `base_footprint`
- `shell_link`
- `rplidar_link`
- `oakd_link`
- `oakd_rgb_camera_frame`
- `oakd_rgb_camera_optical_frame`
- `oakd_left_camera_frame`
- `oakd_right_camera_frame`

Typical meaning:

- `odom`: local odometry frame
- `base_link`: main body frame of the robot
- `base_footprint`: projection of the robot base on the ground plane
- `rplidar_link`: lidar frame
- `oakd_*`: OAK-D camera frames

## A typical dynamic transform

On your robot, `/tf` contains transforms such as:

- `odom -> base_link`
- `odom -> base_footprint`

This is the robot pose estimate changing as the robot moves.

That is why TF is often the easiest way to answer:

- where is the robot now?
- which way is it facing?

## A typical static transform

On your robot, `/tf_static` contains transforms such as:

- `base_link -> shell_link`
- `shell_link -> rplidar_link`
- `shell_link -> oakd_camera_bracket`
- `oakd_camera_bracket -> oakd_link`

These describe where the fixed hardware parts are mounted on the robot.

## Where the TurtleBot4 TF comes from

On the running robot, the main TF publishers are:

- `create3_repub`
- `robot_state_publisher`

Practical split:

- `create3_repub` provides motion-related transforms such as odometry
- `robot_state_publisher` publishes transforms from the robot model and joint state information

## Useful commands

List the TF topics:

```sh
ros2 topic list | rg '^/tf$|^/tf_static$'
```

See one live dynamic transform message:

```sh
ros2 topic echo --once /tf
```

See one static transform message:

```sh
ros2 topic echo --once /tf_static
```

Inspect the transform from `odom` to `base_link`:

```sh
ros2 run tf2_ros tf2_echo odom base_link
```

Inspect the transform from `base_link` to the lidar:

```sh
ros2 run tf2_ros tf2_echo base_link rplidar_link
```

Inspect the transform from `base_link` to the camera:

```sh
ros2 run tf2_ros tf2_echo base_link oakd_link
```

Inspect TF endpoint info:

```sh
ros2 topic info -v /tf
ros2 topic info -v /tf_static
```

## Reading the transform

A transform contains:

- translation: `x`, `y`, `z`
- rotation: quaternion `x`, `y`, `z`, `w`

Example meaning of `odom -> base_link`:

- translation tells you where the robot is in the `odom` frame
- rotation tells you how the robot is oriented in the `odom` frame

## TF and RViz

In RViz, you usually set a fixed frame such as:

- `odom`
- `base_link`

Examples:

- use `odom` when you want to see the robot moving in its local odometry frame
- use `base_link` when you want to inspect sensors relative to the robot body

If RViz shows errors such as missing transforms, check:

```sh
ros2 topic echo --once /tf
ros2 topic echo --once /tf_static
ros2 run tf2_ros tf2_echo odom base_link
```

## TF vs `/odom`

`/odom` is a topic carrying odometry messages:

```sh
nav_msgs/msg/Odometry
```

TF provides the frame relationship:

- `odom -> base_link`

They are related, but they are not the same thing:

- `/odom` is a message stream with pose and twist
- TF is the frame tree used by the rest of ROS 2 to place data in space


## How encoder values become `/odom`

For a differential-drive robot such as the TurtleBot4, odometry is estimated from wheel encoder motion over time.

The wheel encoders measure how much the left and right wheels have rotated.

From that, the robot computes how far each wheel traveled:

- `d_left = wheel_radius * left_wheel_rotation`
- `d_right = wheel_radius * right_wheel_rotation`

Then the robot estimates base motion:

- linear travel: `d = (d_left + d_right) / 2`
- heading change: `dtheta = (d_right - d_left) / wheel_separation`

Using that, the pose is updated:

- `x_new = x + d * cos(theta)`
- `y_new = y + d * sin(theta)`
- `theta_new = theta + dtheta`

This repeated integration over time produces the odometry estimate.

That estimate is then published as:

- the `/odom` topic
- the TF transform `odom -> base_link`

So the practical chain is:

1. wheel encoders measure wheel rotation
2. wheel rotation is converted into left and right wheel travel
3. differential-drive kinematics converts that into robot translation and rotation
4. integration over time gives pose and velocity
5. ROS publishes the result as `/odom`

Important limitation:

- `/odom` is only an estimate
- wheel slip and small errors accumulate over time
- therefore `odom` is locally useful but it drifts and is not a global ground-truth position
