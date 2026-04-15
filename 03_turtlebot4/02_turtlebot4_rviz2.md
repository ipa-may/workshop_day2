# Hands-on Rviz2 with turtlebot4

Start rviz2:

```sh
rviz2
```

You can add RobotModel, Camera, LaserScan.

## OAKD Camera

When the robot is docked for a longer time, the camera may be deactivated to save the battery.

To reactivate the camera, it is necessary to ssh to the turltebot4 and launch the camera node:

```sh
ros2 launch turtlebot4_bringup oakd.launch.py
```

## RobotModel

`RobotModel` is an RViz display that renders the 3D model of the robot.

For TurtleBot4, it uses:

- `/robot_description` for the URDF model
- `/tf` and `/tf_static` for the transforms between robot links
- `/joint_states` to update movable joints

This lets RViz show the TurtleBot4 body, wheels, lidar, camera, and the relative positions of all links and frames.

Typical use:

- verify that the robot model is loaded correctly
- check whether TF is available and consistent
- see whether joints and frames move as expected

Useful commands:

```sh
ros2 topic echo --once /robot_description
ros2 topic echo --once /joint_states
ros2 topic echo --once /tf
ros2 topic echo --once /tf_static
```

## LaserScan

`LaserScan` is an RViz display for 2D lidar measurements. On your TurtleBot4, this comes from the `/scan` topic.

Topic:

```sh
/scan
```

Message type:

```sh
sensor_msgs/msg/LaserScan
```

A `LaserScan` message contains a sequence of distance measurements in meters, each associated with an angle. RViz draws these measurements around the robot so you can see nearby walls, obstacles, and open space.

Typical use:

- confirm that the lidar is publishing
- visualize obstacles around the robot
- check whether scan data aligns with the robot TF frames

Useful commands:

```sh
ros2 topic echo --once /scan
ros2 topic hz /scan
ros2 topic info -v /scan
```

In RViz, set the `LaserScan` topic to `/scan`. If no points are visible, check the RViz fixed frame and make sure TF is available.
