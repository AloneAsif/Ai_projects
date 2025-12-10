---
sidebar_position: 6
---

# Ch 6 — Gazebo Physics Simulation

## Understanding Physics in Robotics Simulation

Physics simulation forms the foundation of realistic robotic behavior in digital twins. Gazebo provides a sophisticated physics engine that accurately models real-world physical interactions, enabling developers to test robot behaviors in environments that closely match physical reality.

The physics engine in Gazebo handles:
- **Gravitational Forces**: Realistic weight and fall behavior
- **Collision Detection**: Accurate contact between robot and environment
- **Friction Modeling**: Realistic surface interactions
- **Joint Dynamics**: Proper mechanical constraints and movement
- **Mass Properties**: Inertia, center of mass, and dynamic behavior

## Setting Up Physics Parameters

### Gravity Configuration
Gravity is the fundamental force that affects all objects in the simulation. For humanoid robots, proper gravity configuration is essential for realistic movement and balance:

```xml
<!-- In world file -->
<world name="humanoid_world">
  <physics type="ode">
    <gravity>0 0 -9.8</gravity>
    <max_step_size>0.001</max_step_size>
    <real_time_factor>1.0</real_time_factor>
    <real_time_update_rate>1000</real_time_update_rate>
  </physics>
</world>
```

The gravity vector `[0, 0, -9.8]` represents Earth's gravitational acceleration in the negative Z direction (downward), with units in m/s².

### Friction Parameters
Friction determines how objects interact with surfaces. For humanoid robots, friction is crucial for walking, grasping, and maintaining balance:

```xml
<!-- In URDF/SDF model -->
<collision name="foot_collision">
  <surface>
    <friction>
      <ode>
        <mu>0.7</mu>  <!-- Static friction coefficient -->
        <mu2>0.7</mu2> <!-- Dynamic friction coefficient -->
        <slip1>0.0</slip1>
        <slip2>0.0</slip2>
      </ode>
    </friction>
  </surface>
</collision>
```

### Mass and Inertia Properties
Proper mass distribution is essential for realistic humanoid movement:

```xml
<!-- In URDF model -->
<link name="torso">
  <inertial>
    <mass value="10.0"/>
    <inertia ixx="0.1" ixy="0.0" ixz="0.0"
             iyy="0.1" iyz="0.0" izz="0.1"/>
  </inertial>
</link>
```

## Collision Detection and Contact Sensors

### Collision Models
Gazebo supports various collision geometries for accurate interaction modeling:

- **Primitive Shapes**: Box, sphere, cylinder for simple collisions
- **Mesh Collisions**: Complex geometries for detailed interactions
- **Compound Collisions**: Multiple shapes combined for complex links

### Contact Sensors
Contact sensors detect when two objects touch, enabling tactile feedback and collision response:

```xml
<!-- In SDF model -->
<sensor name="contact_sensor" type="contact">
  <always_on>true</always_on>
  <update_rate>30</update_rate>
  <contact>
    <collision>link_collision_name</collision>
  </contact>
  <plugin filename="libgazebo_ros_contact.so" name="contact_plugin">
    <ros>
      <namespace>contact_sensor</namespace>
      <topic>~/contacts</topic>
    </ros>
  </plugin>
</sensor>
```

## Spawning Humanoid Robots in Gazebo

### URDF Integration
Humanoid robots are typically described using URDF (Unified Robot Description Format), which Gazebo can directly load:

```xml
<!-- Launch file example -->
<launch>
  <node name="robot_state_publisher" pkg="robot_state_publisher"
        exec="robot_state_publisher" output="screen">
    <param name="robot_description" value="$(var robot_description)"/>
  </node>

  <node name="spawn_entity" pkg="gazebo_ros" exec="spawn_entity.py"
        args="-topic robot_description -entity humanoid_robot"/>
</launch>
```

### Joint Control and Actuation
Gazebo supports various joint control methods for realistic humanoid actuation:

```yaml
# Controller configuration
controller_manager:
  ros__parameters:
    update_rate: 1000  # Hz

    humanoid_controller:
      type: position_controllers/JointGroupPositionController

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster
```

## Practical Exercise: Making a Humanoid Stand Stably

Let's implement a simple controller to make a humanoid robot maintain a stable standing pose:

```python
#!/usr/bin/env python3
# stable_pose_controller.py

import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64MultiArray
from sensor_msgs.msg import JointState
import math

class StablePoseController(Node):
    def __init__(self):
        super().__init__('stable_pose_controller')

        # Joint position publisher
        self.joint_pub = self.create_publisher(
            Float64MultiArray,
            '/position_commands',
            10
        )

        # Joint state subscriber
        self.joint_sub = self.create_subscription(
            JointState,
            '/joint_states',
            self.joint_callback,
            10
        )

        # Target joint positions for stable standing
        self.target_positions = {
            'left_hip_joint': 0.0,
            'left_knee_joint': 0.0,
            'left_ankle_joint': 0.0,
            'right_hip_joint': 0.0,
            'right_knee_joint': 0.0,
            'right_ankle_joint': 0.0,
            'torso_joint': 0.0,
            # Add more joints as needed
        }

        # Control loop timer
        self.timer = self.create_timer(0.01, self.control_loop)  # 100 Hz

    def joint_callback(self, msg):
        # Store current joint states
        self.current_positions = dict(zip(msg.name, msg.position))

    def control_loop(self):
        # Create position command
        cmd = Float64MultiArray()
        cmd.data = [self.target_positions.get(joint, 0.0)
                   for joint in self.target_positions.keys()]

        self.joint_pub.publish(cmd)

def main(args=None):
    rclpy.init(args=args)
    controller = StablePoseController()
    rclpy.spin(controller)
    controller.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Physics Validation and Troubleshooting

### Common Physics Issues
- **Robot falling through ground**: Check collision geometries and surface properties
- **Unstable joints**: Verify joint limits, damping, and stiffness parameters
- **Excessive oscillation**: Adjust solver parameters and joint damping

### Physics Parameter Tuning
```yaml
# Physics parameters for humanoid simulation
physics_config:
  solver_type: "ode"  # or "bullet", "dart"
  step_size: 0.001    # Smaller for more accuracy
  cfm: 0.000001       # Constraint Force Mixing
  erp: 0.2           # Error Reduction Parameter
  max_contacts: 20    # Maximum contacts per collision
```

## Performance Considerations

### Real-time Factor
Monitor the real-time factor to ensure simulation runs at realistic speeds:
- Real-time factor = 1.0: Simulation matches real time
- Real-time factor < 1.0: Simulation slower than real time
- Real-time factor > 1.0: Simulation faster than real time

### Optimization Strategies
- Simplify collision geometries where possible
- Use appropriate update rates for different components
- Limit the number of active physics objects
- Optimize joint limits and constraints

## Summary

This chapter covered the fundamentals of physics simulation in Gazebo, focusing on gravity, friction, collision detection, and humanoid robot spawning. We explored practical implementation of a stable pose controller and discussed physics validation techniques. In the next chapter, we'll dive into sensor simulation to enable perception capabilities in our digital twin.