# 02_urdf_tf

The goal of workshop `02_urdf_tf` is to understand how a robot is described in ROS 2
and how its coordinate frames are connected.

Hands-on:
- [Hands-on_urdf.md](/home/ros2-04/WS_WORKSHOP/github_repo/workshop_day2/02_urdf_tf/Hands-on_urdf.md)

## What is a URDF?

URDF stands for Unified Robot Description Format.
It is an XML-based format used to describe the structure of a robot.

A URDF usually defines:
- links, which represent robot parts such as a base, wheel, or arm segment
- joints, which describe how these parts are connected
- visual properties, so the robot can be displayed in tools such as RViz
- collision properties, so the robot can be used for planning and simulation

In short:
- links are the rigid parts
- joints connect the links
- together they describe the robot model

Example of tutorials:

## About Xacro

Xacro stands for XML Macros.
It is a tool that helps write URDF files in a cleaner and more reusable way.

Instead of repeating the same XML many times, Xacro lets you use:
- variables
- macros
- parameters
- reusable blocks

This is useful because robot descriptions can become large very quickly.
With Xacro, the model is easier to maintain and update than writing one long URDF by hand.

In practice:
- you usually edit a `.xacro` file
- Xacro then generates the final URDF

## What is TF?

TF is the ROS 2 system for keeping track of coordinate frames and the transforms between them.
It tells ROS where one part of the robot is relative to another part.

For example, TF can describe:
- where the camera is relative to the base
- where the laser scanner is relative to the robot
- where the robot is relative to the map

This is important because many ROS tools need to know how frames are connected.
RViz, navigation, robot_state_publisher, and sensor processing all depend on correct TF data.

## URDF and TF together

URDF describes the robot structure.
TF describes the spatial relationship between the robot frames.

In many ROS 2 systems:
- the robot model is written in URDF or Xacro
- `robot_state_publisher` reads that model
- it then publishes the TF tree for the robot

This means URDF defines the robot, and TF makes that structure available at runtime.

## Useful commands

`ros2 topic list`
- Shows available topics.

`ros2 topic echo /robot_description`
- Prints the robot description if it is being published.

`ros2 run tf2_tools view_frames`
- Generates a frame tree overview of the current TF setup.

`ros2 run tf2_ros tf2_echo <source_frame> <target_frame>`
- Shows the transform between two frames.

`ros2 run robot_state_publisher robot_state_publisher`
- Publishes TF frames from a robot description.

