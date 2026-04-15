# Hands on Docker for ROS 2

https://github.com/ipa-may/docker_ros2_tutorial

1. Clone the repo (here no need for a specific filesystem structure)
2. From the root of the repository: build and run a container using docker compose

Example for the realsense camera:
```sh
# Build
docker compose -f 03_ros2_realsense/compose.ros2_camera_jazzy.yaml build

# Start
docker compose -f 03_ros2_realsense/compose.ros2_camera_jazzy.yaml up

# Stop
docker compose -f 03_ros2_realsense/compose.ros2_camera_jazzy.yaml down
```