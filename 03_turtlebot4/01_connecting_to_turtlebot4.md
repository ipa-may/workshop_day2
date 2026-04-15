# Connecting To TurtleBot4

## What the `ros2 daemon` is

The `ros2 daemon` is a local background process used by the `ros2` command line tools.

Its job is to:

- discover ROS 2 nodes, topics, services, and actions
- cache the graph information locally
- make commands such as `ros2 topic list`, `ros2 node list`, and `ros2 action list` faster

It is not a required part of a running robot application. The robot nodes can keep running without it. It mainly exists to help the CLI tools.

Useful commands:

```sh
ros2 daemon status
ros2 daemon stop
ros2 daemon start
```

In practice, when discovery settings change, restarting the daemon is often useful:

```sh
ros2 daemon stop
ros2 daemon start
```

## Simple Discovery with Fast DDS

Fast DDS supports a normal decentralized discovery mode, often called simple discovery.

In this mode:

- each participant announces itself on the network
- discovery usually relies on multicast on the local network
- nodes find each other automatically if the network allows it

Advantages:

- simple to use
- no central discovery server to maintain
- good for small local networks and quick setups

Inconvenients:

- more discovery traffic on the network
- multicast may not work well on some Wi-Fi, VLAN, VPN, or routed networks
- scaling to many participants can become noisy and less predictable

## Discovery Server with Fast DDS

Fast DDS also supports a discovery server architecture.

In this mode:

- participants do not rely on plain multicast-based peer discovery
- they register with one or more discovery servers
- the discovery server helps participants find each other

Advantages:

- less discovery traffic on the network
- more controlled and predictable discovery
- often works better than multicast on restricted or more complex networks
- useful when laptop and robot are not on a clean local multicast-friendly network

Inconvenients:

- extra configuration is required
- a discovery server process must be running
- if the server is unreachable, discovery fails or becomes incomplete
- debugging can be more confusing than simple discovery

## Simple Discovery vs Discovery Server

Main difference:

- simple discovery is decentralized
- discovery server is centralized or semi-centralized

Comparison:

- simple discovery: easier initial setup
- discovery server: better control and usually better behavior on difficult networks
- simple discovery: more multicast traffic
- discovery server: more explicit unicast-style configuration
- simple discovery: tools usually work with less special handling
- discovery server: CLI tools may need extra help, especially the `ros2 daemon`

For the TurtleBot4 setup in this workspace, the discovery server approach is being used because it is more reliable for laptop-to-robot communication on the given network.

## Environment variable `ROS_DISCOVERY_SERVER`

`ROS_DISCOVERY_SERVER` is an environment variable used by Fast DDS clients.

It tells a ROS 2 process where the Fast DDS discovery server is running.

Example:

```sh
export ROS_DISCOVERY_SERVER=192.168.3.180:11811
```

Meaning:

- `192.168.3.180` is the machine running the discovery server
- `11811` is the discovery server port

If this variable is set, Fast DDS uses the configured discovery server instead of relying only on simple multicast discovery.

In this TurtleBot4 setup, the laptop needs this variable so its ROS 2 processes can discover the robot through the robot's discovery server.

## Environment variable `ROS_SUPER_CLIENT`

`ROS_SUPER_CLIENT` is a Fast DDS setting used together with the discovery server mechanism.

When enabled, the participant behaves as a super client.

Practical meaning:

- a normal discovery server client mainly receives the discovery data it needs for its own matched endpoints
- a super client receives the full discovery graph from the discovery server

This is especially useful for:

- `ros2 topic list`
- `ros2 node list`
- `ros2 action list`
- RViz and other introspection tools

Why this matters:

- without `ROS_SUPER_CLIENT`, the laptop may be able to communicate with some robot topics but still fail to see the full ROS graph
- with `ROS_SUPER_CLIENT=TRUE`, the laptop-side `ros2 daemon` and CLI tools can see the complete topic and node list through the discovery server

Example:

```sh
export ROS_SUPER_CLIENT=TRUE
```

## Why this was needed on the laptop

For this TurtleBot4 connection, the laptop used:

```sh
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export ROS_DOMAIN_ID=2
export ROS_DISCOVERY_SERVER=192.168.3.180:11811
export ROS_SUPER_CLIENT=TRUE
```

Reason:

- `RMW_IMPLEMENTATION=rmw_fastrtps_cpp` makes ROS 2 use Fast DDS
- `ROS_DOMAIN_ID=2` puts laptop and robot in the same ROS domain
- `ROS_DISCOVERY_SERVER=192.168.3.180:11811` points the laptop to the robot discovery server
- `ROS_SUPER_CLIENT=TRUE` allows the laptop tools to see the full ROS graph

Without `ROS_SUPER_CLIENT=TRUE`, it is common to see only a partial graph such as:

- `/parameter_events`
- `/rosout`

even though the robot is actually running many more topics.


## About QoS

To know what QoS a topic has: 
```sh
ros2 topic info -v /hmi/led
```