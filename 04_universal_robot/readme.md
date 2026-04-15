# Hands-on Universal Robot

## Building

1. Cloning
Clone the repo from here : https://github.com/ipa-may/rob4_atc_2026_ur_examples into a correct ROS 2 workspace.

You can create another workspace with an `src` folder in it. Inside you clone the repo.

You will need to checkout to the branch `ur5e_real`.

```sh
git clone -b ur5e_real https://github.com/ipa-may/rob4_atc_2026_ur_examples.git
```

2. Getting the upstream dependency repo (also cloning)

This repo contains a .repos file. It specifies the upstream dependency repos. You can clone the specified repo in the `/src` folder.

For that from the `/src` folder:
```sh
vcs import . < rob4_atc_2026_ur_examples/dependencies.repos 
```

(If you want to learn more about vcs tool: https://github.com/dirk-thomas/vcstool)

If you have troubles with vcs tools, you can directly clone the dependency repo :

```sh
git clone -b ros2 https://github.com/fzi-forschungszentrum-informatik/cartesian_controllers.git
```

This repo is the FZI cartesian controllers: https://github.com/fzi-forschungszentrum-informatik/cartesian_controllers

4. rosdep 

Again, don't forget rosdep from the workspace folder:

```sh
rosdep install --from-paths ./ --ignore-src -y
```

5. Building

Now we can build:

```sh
colcon build --packages-skip cartesian_controller_simulation cartesian_controller_tests --cmake-args -DCMAKE_BUILD_TYPE=Release
```

```sh
colcon build --symlink-install --packages-up-to ur3e_ros2_cartesian_control_scripts_examples
```

```sh
colcon build --symlink-install --packages-up-to ur3e_ros2_control_scripts_examples
```



## Running

```sh
ros2 launch ur_atc_robot_cell_control start_robot.launch.py ur_type:=ur5e robot_ip:=192.168.1.10
```

## Using ros2 control CLI

List the controllers and see what
```sh
ros2 control list_controllers
```

List the hardware interfaces

```sh
ros2 control list_hardware_interfaces
```

Switch the controller:

```sh
ros2 control switch_controllers --deactivate <controller1-to-deactivate> --activate <controller2-to-activate> <controller3-to-activate>
```

What we want