# PlotJuggler with TurtleBot4

## What PlotJuggler is

PlotJuggler is a tool to visualize ROS 2 topic values over time.

It is useful when you want to inspect:

- robot velocity
- battery behavior
- IMU signals
- wheel currents
- odometry values

It is much easier for time-series inspection than repeatedly using `ros2 topic echo`.

## Start PlotJuggler

If PlotJuggler is installed with ROS 2 support, start it with:

```sh
ros2 run plotjuggler plotjuggler
```

Before starting it, make sure the TurtleBot4 ROS environment is active:

```sh
source ~/.bashrc
ros2 daemon stop
ros2 daemon start
```

## Connect PlotJuggler to live ROS 2 topics

Inside PlotJuggler:

1. Open the ROS 2 streaming plugin
2. Select the topics you want to monitor
3. Start streaming

After that, you can drag signals into the plot area.

## Good TurtleBot4 topics to plot

Useful topics from your TurtleBot4:

- `/odom`
- `/imu`
- `/battery_state`
- `/wheel_status`
- `/dock_status`
- `/joint_states`
- `/cmd_vel`
- `/cmd_vel_unstamped`

## Example 1: Plot robot linear and angular velocity

Topic:

```sh
/odom
```

Useful fields:

- `/odom/twist/twist/linear/x`
- `/odom/twist/twist/angular/z`

Why:

- check whether the robot is really moving when you send `/cmd_vel`
- compare commanded motion with estimated motion

Typical use:

- send a forward command
- plot `/odom/twist/twist/linear/x`
- send a rotation command
- plot `/odom/twist/twist/angular/z`

## Example 2: Compare command vs actual motion

Topics:

```sh
/cmd_vel_unstamped
/odom
```

Useful fields:

- `/cmd_vel_unstamped/linear/x`
- `/cmd_vel_unstamped/angular/z`
- `/odom/twist/twist/linear/x`
- `/odom/twist/twist/angular/z`

Why:

- see whether the robot follows the command
- detect lag, clipping, or reduced motion

This is one of the most useful PlotJuggler comparisons for TurtleBot4.

## Example 3: Plot odometry position

Topic:

```sh
/odom
```

Useful fields:

- `/odom/pose/pose/position/x`
- `/odom/pose/pose/position/y`

Why:

- see how the robot position evolves over time
- check whether the robot is drifting while driving

Note:

- `/odom` is local odometry and can drift over time

## Example 4: Plot battery data

Topic:

```sh
/battery_state
```

Useful fields:

- `/battery_state/percentage`
- `/battery_state/voltage`
- `/battery_state/current`

Why:

- watch battery charge level
- see charging and discharging behavior
- check current draw during movement or docking

Example interpretation:

- positive or negative current trends can help distinguish charging from discharging
- voltage sag during motion can show load on the system

## Example 5: Plot IMU data

Topic:

```sh
/imu
```

Useful fields:

- `/imu/angular_velocity/z`
- `/imu/linear_acceleration/x`
- `/imu/linear_acceleration/y`
- `/imu/linear_acceleration/z`

Why:

- inspect robot turns
- look for vibration or noise
- compare IMU rotation with odometry rotation

A useful comparison is:

- `/imu/angular_velocity/z`
- `/odom/twist/twist/angular/z`

## Example 6: Plot wheel status

Topic:

```sh
/wheel_status
```

Useful fields:

- `/wheel_status/current_ma_left`
- `/wheel_status/current_ma_right`
- `/wheel_status/pwm_left`
- `/wheel_status/pwm_right`
- `/wheel_status/wheels_enabled`

Why:

- inspect motor effort
- compare left and right wheel load
- detect whether wheels are enabled

This is especially useful when the robot is commanded to move but does not behave as expected.

## Example 7: Plot docking state

Topic:

```sh
/dock_status
```

Useful fields:

- `/dock_status/dock_visible`
- `/dock_status/is_docked`

Why:

- observe docking and undocking transitions
- verify whether the dock is detected before a dock action

## Example 8: Plot joystick or teleop commands

Topics:

```sh
/joy
/cmd_vel
/cmd_vel_unstamped
```

Why:

- verify whether joystick input is producing motion commands
- correlate user input with robot movement

Useful fields:

- joystick axes from `/joy`
- velocity commands from `/cmd_vel` or `/cmd_vel_unstamped`

## Topics less convenient to plot

Some TurtleBot4 topics are less convenient in PlotJuggler:

- `/scan` because it contains arrays of ranges
- `/tf` because it contains many transforms in arrays
- `/robot_description` because it is not a numeric time series

You can still inspect those topics with other ROS 2 tools, but they are not the best first choice for PlotJuggler.

## Practical workflow

A good basic PlotJuggler session for TurtleBot4 is:

1. plot `/cmd_vel_unstamped/linear/x`
2. plot `/odom/twist/twist/linear/x`
3. plot `/cmd_vel_unstamped/angular/z`
4. plot `/odom/twist/twist/angular/z`
5. plot `/battery_state/voltage`
6. plot `/wheel_status/current_ma_left`
7. plot `/wheel_status/current_ma_right`

This gives a quick view of:

- commanded motion
- actual estimated motion
- battery behavior
- motor load

## Summary

PlotJuggler is useful on TurtleBot4 when you want to understand behavior over time instead of just reading single messages.

The best topics to start with are:

- `/odom`
- `/cmd_vel_unstamped`
- `/imu`
- `/battery_state`
- `/wheel_status`

These are usually enough to debug motion, teleoperation, battery behavior, and basic robot state.
