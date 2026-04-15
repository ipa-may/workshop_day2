# Subscribe To TurtleBot4

These examples assume you are on the laptop and your shell already has the TurtleBot4 discovery settings:

```bash
source ~/.bashrc
ros2 daemon stop
ros2 daemon start
```

## Common Topics To Read

| Topic | Type | What You Get |
| --- | --- | --- |
| `/battery_state` | `sensor_msgs/msg/BatteryState` | Battery percentage, voltage, current |
| `/dock_status` | `irobot_create_msgs/msg/DockStatus` | Whether the dock is visible and whether the robot is docked |
| `/imu` | `sensor_msgs/msg/Imu` | Orientation, angular velocity, linear acceleration |
| `/joint_states` | `sensor_msgs/msg/JointState` | Wheel joint positions and velocities |
| `/odom` | `nav_msgs/msg/Odometry` | Base pose and twist estimate |
| `/scan` | `sensor_msgs/msg/LaserScan` | 2D lidar scan |
| `/wheel_status` | `irobot_create_msgs/msg/WheelStatus` | Wheel currents, PWM, and enabled state |
| `/hmi/buttons` | `turtlebot4_msgs/msg/UserButton` | 4 TurtleBot4 user buttons |
| `/interface_buttons` | `irobot_create_msgs/msg/InterfaceButtons` | Create 3 faceplate buttons |
| `/joy` | `sensor_msgs/msg/Joy` | Joystick axes and buttons |
| `/tf` | `tf2_msgs/msg/TFMessage` | Dynamic transforms |
| `/tf_static` | `tf2_msgs/msg/TFMessage` | Static transforms |

## ROS 2 CLI Commands

### Show one message

Battery:

```bash
ros2 topic echo --once /battery_state
```

Dock status:

```bash
ros2 topic echo --once /dock_status
```

Odometry:

```bash
ros2 topic echo --once /odom
```

Laser scan:

```bash
ros2 topic echo --once /scan
```

Wheel status:

```bash
ros2 topic echo --once /wheel_status
```

### Stream messages continuously

```bash
ros2 topic echo /imu
ros2 topic echo /joint_states
ros2 topic echo /hmi/buttons
ros2 topic echo /interface_buttons
ros2 topic echo /joy
ros2 topic echo /tf
ros2 topic echo /tf_static
```

### Check update rates

```bash
ros2 topic hz /scan
ros2 topic hz /odom
ros2 topic hz /imu
```

### Inspect topic types and endpoint info

```bash
ros2 topic type /scan
ros2 topic info -v /scan
ros2 interface show sensor_msgs/msg/LaserScan
```

## Python Snippets

### Read battery, odometry, dock status, wheel status, and lidar

```python
#!/usr/bin/env python3
import math

import rclpy
from irobot_create_msgs.msg import DockStatus, WheelStatus
from nav_msgs.msg import Odometry
from rclpy.node import Node
from rclpy.qos import QoSDurabilityPolicy, QoSProfile, QoSReliabilityPolicy
from sensor_msgs.msg import BatteryState, LaserScan


class TurtleBot4StateSubscriber(Node):
    def __init__(self):
        super().__init__('turtlebot4_state_subscriber')

        reliable_qos = QoSProfile(
            depth=10,
            reliability=QoSReliabilityPolicy.RELIABLE,
            durability=QoSDurabilityPolicy.VOLATILE,
        )
        latched_qos = QoSProfile(
            depth=10,
            reliability=QoSReliabilityPolicy.RELIABLE,
            durability=QoSDurabilityPolicy.TRANSIENT_LOCAL,
        )

        self.create_subscription(BatteryState, '/battery_state', self.battery_cb, latched_qos)
        self.create_subscription(Odometry, '/odom', self.odom_cb, latched_qos)
        self.create_subscription(DockStatus, '/dock_status', self.dock_cb, latched_qos)
        self.create_subscription(WheelStatus, '/wheel_status', self.wheel_cb, latched_qos)
        self.create_subscription(LaserScan, '/scan', self.scan_cb, reliable_qos)

    def battery_cb(self, msg: BatteryState):
        percent = 100.0 * msg.percentage if not math.isnan(msg.percentage) else float('nan')
        self.get_logger().info(
            f'battery: {percent:.1f}%  voltage: {msg.voltage:.2f} V  current: {msg.current:.2f} A'
        )

    def odom_cb(self, msg: Odometry):
        x = msg.pose.pose.position.x
        y = msg.pose.pose.position.y
        vx = msg.twist.twist.linear.x
        wz = msg.twist.twist.angular.z
        self.get_logger().info(f'odom: x={x:.3f} y={y:.3f} vx={vx:.3f} wz={wz:.3f}')

    def dock_cb(self, msg: DockStatus):
        self.get_logger().info(
            f'dock: visible={msg.dock_visible} docked={msg.is_docked}'
        )

    def wheel_cb(self, msg: WheelStatus):
        self.get_logger().info(
            'wheel: '
            f'left_current={msg.current_ma_left} mA '
            f'right_current={msg.current_ma_right} mA '
            f'enabled={msg.wheels_enabled}'
        )

    def scan_cb(self, msg: LaserScan):
        valid_ranges = [r for r in msg.ranges if msg.range_min <= r <= msg.range_max]
        if valid_ranges:
            self.get_logger().info(
                f'scan: points={len(msg.ranges)} nearest={min(valid_ranges):.3f} m'
            )
        else:
            self.get_logger().info(f'scan: points={len(msg.ranges)} no valid returns')


def main():
    rclpy.init()
    node = TurtleBot4StateSubscriber()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### Read TurtleBot4 and Create 3 buttons

`/hmi/buttons` is best-effort on this robot, so the subscriber uses a best-effort QoS profile.

```python
#!/usr/bin/env python3
import rclpy
from irobot_create_msgs.msg import InterfaceButtons
from rclpy.node import Node
from rclpy.qos import QoSDurabilityPolicy, QoSProfile, QoSReliabilityPolicy
from turtlebot4_msgs.msg import UserButton


class ButtonSubscriber(Node):
    def __init__(self):
        super().__init__('tb4_button_subscriber')

        best_effort_qos = QoSProfile(
            depth=10,
            reliability=QoSReliabilityPolicy.BEST_EFFORT,
            durability=QoSDurabilityPolicy.VOLATILE,
        )
        reliable_qos = QoSProfile(
            depth=10,
            reliability=QoSReliabilityPolicy.RELIABLE,
            durability=QoSDurabilityPolicy.TRANSIENT_LOCAL,
        )

        self.create_subscription(UserButton, '/hmi/buttons', self.user_buttons_cb, best_effort_qos)
        self.create_subscription(
            InterfaceButtons,
            '/interface_buttons',
            self.interface_buttons_cb,
            reliable_qos,
        )

    def user_buttons_cb(self, msg: UserButton):
        self.get_logger().info(f'user buttons: {list(msg.button)}')

    def interface_buttons_cb(self, msg: InterfaceButtons):
        self.get_logger().info(
            'interface buttons: '
            f'button_1={msg.button_1.is_pressed} '
            f'power={msg.button_power.is_pressed} '
            f'button_2={msg.button_2.is_pressed}'
        )


def main():
    rclpy.init()
    node = ButtonSubscriber()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```
