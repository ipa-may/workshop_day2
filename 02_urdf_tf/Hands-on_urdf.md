# Hands-on URDF

## Links
Explanation: https://ros-industrial.github.io/ros2_i_training/_source/urdf/cartesian_tutorial.html

code: https://github.com/ipa-may/cartesian_robot_tutorial

## Getting Started
Explanation on cloning the code:

You need to create a `workspace` with a `/src` folder inside.

A correct structure for this exercise looks like:
```txt
my_ws/
├── src/
│   ├── <package-cloned>
```

After cloning, you need to run rosdep from the workspace (`/my_ws`):

```sh
rosdep install --from-paths src --ignore-src -ry
```

Then from the workspace, run colcon build:

```sh
colcon build --symlink-install
```

## Running (two exercices) 
1. First exercice
Running the rviz view for visualizing the urdf:
```sh
ros2 launch cartesian_robot_description cartesian_robot_view.launch.py
```

In Rviz, add the RobotModel. Under `Description Topic` select `/robot_description`.

The `/map` topic does not exist in this scene. Therefore under `Global Options` Display, select near the `Fixed Frame` the `/base_link`.

You can now visualize the gantry system.

Once done, you can close the running ros2 launch.

2. Second exercice

Running Gazebo and Moveit

```sh
ros2 launch cartesian_robot_simulation_gz cartesian_sim_moveit.launch.py
```