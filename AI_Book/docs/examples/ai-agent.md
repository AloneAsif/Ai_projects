---
sidebar_position: 8
---

# AI Agent Example

This example demonstrates a simple AI agent that connects to a ROS system, reads sensor data, makes decisions, and sends commands to the robot.

## AI Agent Code

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

## How to Run

1. Make sure your robot is running and publishing sensor data
2. Run the AI agent: `ros2 run your_package_name ai_agent`

## Expected Behavior

The AI agent will continuously read laser data from the robot and make simple obstacle avoidance decisions. If it detects an obstacle in front of the robot, it will send a command to turn right; otherwise, it will send a command to move forward.

## Key Concepts Demonstrated

- AI agent architecture with perception-action cycle
- Reading sensor data from ROS topics
- Making decisions based on sensor input
- Sending commands to robot through ROS topics
- Timer-based decision-making loop
- Simple obstacle avoidance algorithm