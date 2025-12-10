---
sidebar_position: 4
---

# Ch 3 — Python AI Agent → ROS Bridge

## Introduction to AI Agent Integration

Integrating AI agents with ROS-based robotic systems enables sophisticated decision-making capabilities and autonomous behavior. The bridge between AI agents and ROS systems allows for complex task planning, perception processing, and action execution in a coordinated manner.

## Simple Task-Based Agent

An AI agent in a robotic context typically follows a perception-action cycle:
1. **Perceive**: Read sensor data from ROS topics
2. **Process**: Apply AI algorithms to interpret data and make decisions
3. **Act**: Send commands to ROS nodes for robot execution
4. **Monitor**: Observe the results and adjust behavior accordingly

### Basic AI Agent Structure
```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from geometry_msgs.msg import Twist
from sensor_msgs.msg import LaserScan
import numpy as np

class AIAgent(Node):
    def __init__(self):
        super().__init__('ai_agent')

        # Publishers for sending robot commands
        self.cmd_vel_publisher = self.create_publisher(Twist, '/cmd_vel', 10)

        # Subscribers for receiving sensor data
        self.laser_subscriber = self.create_subscription(
            LaserScan, '/scan', self.laser_callback, 10)
        self.odom_subscriber = self.create_subscription(
            String, '/robot_status', self.status_callback, 10)

        # Timer for the AI decision-making loop
        self.timer = self.create_timer(0.1, self.decision_loop)

        # Internal state for the AI agent
        self.laser_data = None
        self.robot_status = "idle"
        self.goal_position = None

    def laser_callback(self, msg):
        """Process laser scan data from the robot"""
        self.laser_data = msg.ranges
        self.get_logger().info(f'Laser data received: {len(msg.ranges)} points')

    def status_callback(self, msg):
        """Process robot status updates"""
        self.robot_status = msg.data

    def decision_loop(self):
        """Main AI decision-making function"""
        if self.laser_data is not None:
            # Example: Simple obstacle avoidance
            command = self.make_decision()
            self.cmd_vel_publisher.publish(command)

    def make_decision(self):
        """AI decision-making logic"""
        cmd = Twist()

        # Simple obstacle avoidance: if something is close in front, turn
        if self.laser_data:
            front_distance = min(self.laser_data[300:420])  # Front-facing range

            if front_distance < 1.0:  # If obstacle closer than 1 meter
                cmd.linear.x = 0.0
                cmd.angular.z = 0.5  # Turn right
            else:
                cmd.linear.x = 0.5  # Move forward
                cmd.angular.z = 0.0

        return cmd

def main(args=None):
    rclpy.init(args=args)
    ai_agent = AIAgent()
    rclpy.spin(ai_agent)
    ai_agent.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Sending Robot Commands

The AI agent communicates with the robot through ROS topics and services. Common command types include:

### Velocity Commands
```python
from geometry_msgs.msg import Twist

def send_velocity_command(self, linear_x, angular_z):
    """Send velocity command to robot"""
    cmd = Twist()
    cmd.linear.x = linear_x
    cmd.angular.y = 0.0
    cmd.linear.z = 0.0
    cmd.angular.x = 0.0
    cmd.angular.y = 0.0
    cmd.angular.z = angular_z

    self.cmd_vel_publisher.publish(cmd)
```

### Navigation Goals
```python
from geometry_msgs.msg import PoseStamped

def send_navigation_goal(self, x, y, theta):
    """Send navigation goal to move_base"""
    goal = PoseStamped()
    goal.header.frame_id = "map"
    goal.header.stamp = self.get_clock().now().to_msg()
    goal.pose.position.x = x
    goal.pose.position.y = y
    goal.pose.orientation.z = theta

    self.navigation_publisher.publish(goal)
```

### Service Calls for Complex Actions
```python
from std_srvs.srv import Trigger

def call_robot_service(self, service_name):
    """Call a robot service"""
    client = self.create_client(Trigger, service_name)
    while not client.wait_for_service(timeout_sec=1.0):
        self.get_logger().info(f'Service {service_name} not available, waiting...')

    request = Trigger.Request()
    future = client.call_async(request)
    return future
```

## Reading Sensor Topics

AI agents need to process sensor data to make informed decisions:

### Processing Camera Data
```python
from sensor_msgs.msg import Image
from cv_bridge import CvBridge

class VisionAgent(AIAgent):
    def __init__(self):
        super().__init__()
        self.image_subscriber = self.create_subscription(
            Image, '/camera/image_raw', self.image_callback, 10)
        self.bridge = CvBridge()

    def image_callback(self, msg):
        """Process camera image data"""
        try:
            cv_image = self.bridge.imgmsg_to_cv2(msg, "bgr8")
            # Process image with AI algorithms
            processed_result = self.process_vision(cv_image)
            self.handle_vision_result(processed_result)
        except Exception as e:
            self.get_logger().error(f'Error processing image: {e}')
```

### Processing LIDAR Data
```python
def process_lidar_data(self):
    """Process LIDAR data for navigation and obstacle detection"""
    if self.laser_data:
        # Convert to numpy array for easier processing
        ranges = np.array(self.laser_data)

        # Remove invalid readings (inf, nan)
        valid_ranges = ranges[np.isfinite(ranges)]

        # Calculate statistics
        min_distance = np.min(valid_ranges) if len(valid_ranges) > 0 else float('inf')
        avg_distance = np.mean(valid_ranges) if len(valid_ranges) > 0 else float('inf')

        return {
            'min_distance': min_distance,
            'avg_distance': avg_distance,
            'valid_readings': len(valid_ranges)
        }
```

## Advanced AI Integration Patterns

### Behavior Trees
Behavior trees provide a structured approach to complex AI decision-making:

```python
class BehaviorTreeAgent(AIAgent):
    def __init__(self):
        super().__init__()
        self.current_behavior = "explore"

    def decision_loop(self):
        """Execute behavior tree logic"""
        if self.current_behavior == "explore":
            self.explore_behavior()
        elif self.current_behavior == "avoid_obstacles":
            self.avoid_obstacles_behavior()
        elif self.current_behavior == "return_home":
            self.return_home_behavior()
```

### State Machines
State machines can be used for simpler decision-making:

```python
class StateMachineAgent(AIAgent):
    def __init__(self):
        super().__init__()
        self.state = "idle"
        self.state_transitions = {
            "idle": self.idle_state,
            "moving": self.moving_state,
            "avoiding": self.avoiding_state,
            "reached_goal": self.reached_goal_state
        }

    def decision_loop(self):
        """Execute current state logic"""
        if self.state in self.state_transitions:
            self.state_transitions[self.state]()
```

## Error Handling and Safety

AI agents must include robust error handling:

```python
def safe_command_publisher(self, cmd):
    """Safely publish commands with validation"""
    # Validate command parameters
    if abs(cmd.linear.x) > self.max_linear_speed:
        cmd.linear.x = self.max_linear_speed if cmd.linear.x > 0 else -self.max_linear_speed

    if abs(cmd.angular.z) > self.max_angular_speed:
        cmd.angular.z = self.max_angular_speed if cmd.angular.z > 0 else -self.max_angular_speed

    # Check for safety conditions
    if self.emergency_stop_active:
        cmd.linear.x = 0.0
        cmd.angular.z = 0.0

    self.cmd_vel_publisher.publish(cmd)
```

## Summary

This chapter demonstrated how to connect a Python-based AI agent to a ROS system. We explored patterns for sending robot commands, reading sensor data, and implementing decision-making logic. The AI agent acts as a bridge between high-level AI algorithms and low-level robot control, enabling sophisticated autonomous behavior. In the next chapter, we'll explore URDF for humanoid robot modeling.