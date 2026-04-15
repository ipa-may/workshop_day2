# Action Requests

Actions explained in the ros2 doc:
https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html


These examples assume your laptop is already configured for the TurtleBot4:

```sh
source ~/.bashrc
ros2 daemon stop
ros2 daemon start
```

To see the available actions:

```sh
ros2 action list -t
```

To send an action goal and also see feedback:

```sh
ros2 action send_goal --feedback <action_name> <action_type> "<goal_yaml>"
```

Be careful with any action that moves the robot. Test in a clear area.

## /audio_note_sequence

Action type:

```sh
irobot_create_msgs/action/AudioNoteSequence
```

Play two notes once:

```sh
ros2 action send_goal --feedback /audio_note_sequence irobot_create_msgs/action/AudioNoteSequence \
'{iterations: 1, note_sequence: {notes: [{frequency: 440, max_runtime: {sec: 0, nanosec: 300000000}}, {frequency: 660, max_runtime: {sec: 0, nanosec: 300000000}}], append: false}}'
```

## /dock

Action type:

```sh
irobot_create_msgs/action/Dock
```

Send dock action:

```sh
ros2 action send_goal --feedback /dock irobot_create_msgs/action/Dock '{}'
```

## /drive_arc

Action type:

```sh
irobot_create_msgs/action/DriveArc
```

Drive forward along a left arc:

```sh
ros2 action send_goal --feedback /drive_arc irobot_create_msgs/action/DriveArc \
'{translate_direction: 1, angle: 1.57, radius: 0.5, max_translation_speed: 0.2}'
```

Notes:

- `translate_direction: 1` means forward
- `translate_direction: -1` means backward
- `angle` is in radians
- `radius` is in meters

## /drive_distance

Action type:

```sh
irobot_create_msgs/action/DriveDistance
```

Drive forward 0.5 m:

```sh
ros2 action send_goal --feedback /drive_distance irobot_create_msgs/action/DriveDistance \
'{distance: 0.5, max_translation_speed: 0.2}'
```

Drive backward 0.2 m:

```sh
ros2 action send_goal --feedback /drive_distance irobot_create_msgs/action/DriveDistance \
'{distance: -0.2, max_translation_speed: 0.1}'
```

## /led_animation

Action type:

```sh
irobot_create_msgs/action/LedAnimation
```

Blink all 6 light ring LEDs in green for 3 seconds:

Example 1:
```sh
ros2 action send_goal --feedback /led_animation irobot_create_msgs/action/LedAnimation \
'{animation_type: 1, lightring: {leds: [{red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}], override_system: true}, max_runtime: {sec: 3, nanosec: 0}}'
```

Example 2:
```sh
ros2 action send_goal --feedback /led_animation irobot_create_msgs/action/LedAnimation '{animation_type: 2, lightring: {leds: [{red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 0, blue: 255}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}, {red: 0, green: 255, blue: 0}], override_system: true}, max_runtime: {sec: 3, nanosec: 0}}'
```


Notes:

- `animation_type: 1` is `BLINK_LIGHTS`
- `animation_type: 2` is `SPIN_LIGHTS`

## /navigate_to_position

Action type:

```sh
irobot_create_msgs/action/NavigateToPosition
```

Drive to position `(x=1.0, y=0.0)` in the `odom` frame and finish with heading `0 rad`:

```sh
ros2 action send_goal --feedback /navigate_to_position irobot_create_msgs/action/NavigateToPosition \
'{goal_pose: {header: {frame_id: "odom"}, pose: {position: {x: 1.0, y: 0.0, z: 0.0}, orientation: {x: 0.0, y: 0.0, z: 0.0, w: 1.0}}}, achieve_goal_heading: true, max_translation_speed: 0.2, max_rotation_speed: 1.0}'
```

For a final heading of about `+90 deg`, use quaternion `z: 0.707, w: 0.707`.

You can know the current position of the robot in the /odom frame by echo of the /odom topic:
```sh
ros2 topic echo --once /odom
```

Or by using `tf2_ros` for a more convenient live view:
```sh
ros2 run tf2_ros tf2_echo odom base_link
```

That shows the current robot pose of base_link relative to odom, which is often easier to read than the full /odom message.

## /rotate_angle

Action type:

```sh
irobot_create_msgs/action/RotateAngle
```

Rotate left by 90 degrees:

```sh
ros2 action send_goal --feedback /rotate_angle irobot_create_msgs/action/RotateAngle \
'{angle: 1.57, max_rotation_speed: 1.0}'
```

Rotate right by 90 degrees:

```sh
ros2 action send_goal --feedback /rotate_angle irobot_create_msgs/action/RotateAngle \
'{angle: -1.57, max_rotation_speed: 1.0}'
```


## /undock

Action type:

```sh
irobot_create_msgs/action/Undock
```

Send undock action:

```sh
ros2 action send_goal --feedback /undock irobot_create_msgs/action/Undock '{}'
```

## /wall_follow

Action type:

```sh
irobot_create_msgs/action/WallFollow
```

Follow a wall on the left for 10 seconds:

```sh
ros2 action send_goal --feedback /wall_follow irobot_create_msgs/action/WallFollow \
'{follow_side: 1, max_runtime: {sec: 10, nanosec: 0}}'
```

Follow a wall on the right for 10 seconds:

```sh
ros2 action send_goal --feedback /wall_follow irobot_create_msgs/action/WallFollow \
'{follow_side: -1, max_runtime: {sec: 10, nanosec: 0}}'
```

Notes:

- `follow_side: 1` is `FOLLOW_LEFT`
- `follow_side: -1` is `FOLLOW_RIGHT`
