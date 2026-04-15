# What is a correct ROS 2 workspace?

When you want to clone a git ROS 2 package:

You should create a workspace directory, create an `src/` folder inside it, and clone the repo into `src/`.

A correct structure for this exercise looks like:
```txt
my_ur_ws/
├── src/
│   ├── rob4_atc_2026_ur_examples/
│   └── cartesian_controllers/   # after vcs import or manual clone
```

Here `my_ur_ws` is the workspace root.

You can:
- run `rosdep install ...` from the workspace root my_ur_ws/
- run `colcon build ...` from the workspace root
- source ROS first with `source /opt/ros/jazzy/setup.bash`
- after building, you would typically also source `install/setup.bash` from that workspace