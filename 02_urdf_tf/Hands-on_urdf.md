# Hands-on URDF

Explanation: https://ros-industrial.github.io/ros2_i_training/_source/urdf/cartesian_tutorial.html

code: https://github.com/ipa-may/cartesian_robot_tutorial

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