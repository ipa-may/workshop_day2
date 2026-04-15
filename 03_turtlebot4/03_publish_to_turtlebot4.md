# Topic Publish To TurtleBot4

These examples assume you are on the laptop and your shell already has the TurtleBot4 discovery settings:
```sh
# .bashrc
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export ROS_DOMAIN_ID=2 # same ROS_DOMAIN_ID as the turtlebot4
export ROS_DISCOVERY_SERVER=192.168.3.180:11811 # corresponds to the IP of the 
export ROS_SUPER_CLIENT=TRUE # 
```


```bash
source ~/.bashrc
ros2 daemon stop
ros2 daemon start
```

For anything that moves the base, keep the robot lifted or use a clear area first.

## Topics You Can Send To

| Topic | Type | Purpose |
| --- | --- | --- |
| `/cmd_vel_unstamped` | `geometry_msgs/msg/Twist` | Simplest way to command base velocity |
| `/cmd_vel` | `geometry_msgs/msg/TwistStamped` | Stamped velocity command |
| `/cmd_audio` | `irobot_create_msgs/msg/AudioNoteVector` | Play tones on the robot |
| `/cmd_lightring` | `irobot_create_msgs/msg/LightringLeds` | Control the Create 3 light ring |
| `/hmi/display/message` | `std_msgs/msg/String` | Show a text message on the TurtleBot4 display |
| `/hmi/display` | `turtlebot4_msgs/msg/UserDisplay` | Replace the full TurtleBot4 display state |
| `/hmi/led` | `turtlebot4_msgs/msg/UserLed` | Control user LEDs |
| `/joy/set_feedback` | `sensor_msgs/msg/JoyFeedbackArray` | Send joystick rumble/LED feedback |

## ROS 2 CLI Commands

### Move the robot with `/cmd_vel_unstamped`

Forward at 0.15 m/s:

```bash
ros2 topic pub -r 10 /cmd_vel_unstamped geometry_msgs/msg/Twist \
"{linear: {x: 0.15, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
```

Rotate in place:

```bash
ros2 topic pub -r 10 /cmd_vel_unstamped geometry_msgs/msg/Twist \
"{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.6}}"
```

Stop:

```bash
ros2 topic pub --once /cmd_vel_unstamped geometry_msgs/msg/Twist \
"{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
```

Remember the turtlesim control with the keyboard? What node was it?

package name: turtlesim
exec: turtle_teleop_key

However it publish on the topic `/turtle1/cmd_vel` which is of type `geometry_msgs/msg/Twist`. We want to remap it to topic name `/cmd_vel_unstamped`. This is possible by remapping the topic name while starting the node:

```sh
ros2 run turtlesim turtle_teleop_key --ros-args -r /turtle1/cmd_vel:=/cmd_vel_unstamped
```

Now you can move the turtlebot4 around as if it is a turtlesim.


### Move the robot with `/cmd_vel`

```bash
ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/TwistStamped \
"{header: {frame_id: 'base_link'}, twist: {linear: {x: 0.10, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}}"
```

### Send a text message to the TurtleBot4 display

```bash
ros2 topic pub --once /hmi/display/message std_msgs/msg/String \
"{data: 'Hello from laptop'}"
```


### Control the TurtleBot4 user LED

topic `/hmi/led` is `BEST_EFFORT`.

`led`: `0` = `USER_LED_1`, `1` = `USER_LED_2`  
`color`: `0` = off, `1` = green, `2` = red, `3` = yellow

```bash
ros2 topic pub --once /hmi/led turtlebot4_msgs/msg/UserLed \
"{led: 0, color: 1, blink_period: 500, duty_cycle: 0.5}"
```

Make it stop blinking:
```sh
ros2 topic pub -t 5 -r 5 /hmi/led turtlebot4_msgs/msg/UserLed \
"{led: 0, color: 0, blink_period: 0, duty_cycle: 0.0}"
```

### Send joystick feedback with `/joy/set_feedback`

Rumble channel 0 at full intensity:

```bash
ros2 topic pub --once /joy/set_feedback sensor_msgs/msg/JoyFeedbackArray \
"{array: [{type: 1, id: 0, intensity: 1.0}]}"
```

## Python Snippets

### Publish continuous velocity on `/cmd_vel_unstamped`

```python
#!/usr/bin/env python3
import time

import rclpy
from geometry_msgs.msg import Twist
from rclpy.node import Node


class CmdVelPublisher(Node):
    def __init__(self):
        super().__init__('cmd_vel_unstamped_publisher')
        self.publisher = self.create_publisher(Twist, '/cmd_vel_unstamped', 10)
        self.timer = self.create_timer(0.1, self.publish_cmd)

    def publish_cmd(self):
        msg = Twist()
        msg.linear.x = 0.15
        msg.angular.z = 0.0
        self.publisher.publish(msg)


def main():
    rclpy.init()
    node = CmdVelPublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        stop_msg = Twist()
        node.publisher.publish(stop_msg)
        time.sleep(0.2)
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### Publish stamped velocity on `/cmd_vel`

```python
#!/usr/bin/env python3
import rclpy
from geometry_msgs.msg import TwistStamped
from rclpy.node import Node


class CmdVelStampedPublisher(Node):
    def __init__(self):
        super().__init__('cmd_vel_stamped_publisher')
        self.publisher = self.create_publisher(TwistStamped, '/cmd_vel', 10)
        self.timer = self.create_timer(0.1, self.publish_cmd)

    def publish_cmd(self):
        msg = TwistStamped()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.header.frame_id = 'base_link'
        msg.twist.linear.x = 0.10
        msg.twist.angular.z = 0.0
        self.publisher.publish(msg)


def main():
    rclpy.init()
    node = CmdVelStampedPublisher()
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

### Publish audio once on `/cmd_audio`

```python
#!/usr/bin/env python3
import time

import rclpy
from builtin_interfaces.msg import Duration
from irobot_create_msgs.msg import AudioNote, AudioNoteVector
from rclpy.node import Node


def main():
    rclpy.init()
    node = Node('cmd_audio_once')
    publisher = node.create_publisher(AudioNoteVector, '/cmd_audio', 10)

    time.sleep(1.0)

    note1 = AudioNote()
    note1.frequency = 440
    note1.max_runtime = Duration(sec=0, nanosec=300_000_000)

    note2 = AudioNote()
    note2.frequency = 660
    note2.max_runtime = Duration(sec=0, nanosec=300_000_000)

    msg = AudioNoteVector()
    msg.notes = [note1, note2]
    msg.append = False

    publisher.publish(msg)
    time.sleep(0.2)

    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### Publish light ring, display text, and user LED

```python
#!/usr/bin/env python3
import time

import rclpy
from irobot_create_msgs.msg import LedColor, LightringLeds
from rclpy.node import Node
from std_msgs.msg import String
from turtlebot4_msgs.msg import UserDisplay, UserLed


def main():
    rclpy.init()
    node = Node('tb4_hmi_publishers')

    lightring_pub = node.create_publisher(LightringLeds, '/cmd_lightring', 10)
    display_msg_pub = node.create_publisher(String, '/hmi/display/message', 10)
    display_pub = node.create_publisher(UserDisplay, '/hmi/display', 10)
    led_pub = node.create_publisher(UserLed, '/hmi/led', 10)

    time.sleep(1.0)

    lightring_msg = LightringLeds()
    lightring_msg.leds = [LedColor(red=0, green=255, blue=0) for _ in range(6)]
    lightring_msg.override_system = True
    lightring_pub.publish(lightring_msg)

    display_msg_pub.publish(String(data='Hello from Python'))

    display_msg = UserDisplay()
    display_msg.ip = '192.168.3.180'
    display_msg.battery = '87%'
    display_msg.entries = ['Start', 'Stop', 'Dock', 'Undock', 'Demo']
    display_msg.selected_entry = 1
    display_pub.publish(display_msg)

    led_msg = UserLed()
    led_msg.led = UserLed.USER_LED_1
    led_msg.color = UserLed.COLOR_GREEN
    led_msg.blink_period = 500
    led_msg.duty_cycle = 0.5
    led_pub.publish(led_msg)

    time.sleep(0.2)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
