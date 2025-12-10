---
sidebar_position: 7
---

# Ch 7 — Sensor Simulation (LiDAR, Depth, IMU)

## Introduction to Sensor Simulation

Sensor simulation is a critical component of digital twin technology, enabling robots to perceive their virtual environment with realistic data. In the context of humanoid robotics, accurate sensor simulation allows for comprehensive testing of perception algorithms, navigation systems, and human-robot interaction scenarios without the need for physical hardware.

The key benefits of sensor simulation include:
- **Safe Testing**: Validate perception algorithms without risk to physical sensors
- **Consistent Data**: Reproducible sensor data for algorithm development
- **Cost Efficiency**: Test multiple sensor configurations virtually
- **Edge Case Testing**: Simulate rare or dangerous scenarios safely

## LiDAR Simulation in Gazebo

### LiDAR Sensor Configuration
LiDAR (Light Detection and Ranging) sensors provide 2D or 3D point cloud data for environment mapping and obstacle detection:

```xml
<!-- In SDF model -->
<sensor name="lidar_sensor" type="ray">
  <ray>
    <scan>
      <horizontal>
        <samples>720</samples>
        <resolution>1</resolution>
        <min_angle>-1.570796</min_angle>  <!-- -90 degrees -->
        <max_angle>1.570796</max_angle>   <!-- 90 degrees -->
      </horizontal>
    </scan>
    <range>
      <min>0.1</min>
      <max>10.0</max>
      <resolution>0.01</resolution>
    </range>
  </ray>
  <plugin filename="libgazebo_ros_ray_sensor.so" name="lidar_plugin">
    <ros>
      <namespace>lidar</namespace>
      <remapping>~/out:=scan</remapping>
    </ros>
    <output_type>sensor_msgs/LaserScan</output_type>
  </plugin>
  <always_on>true</always_on>
  <update_rate>10</update_rate>
</sensor>
```

### LiDAR Data Processing
The simulated LiDAR publishes `sensor_msgs/LaserScan` messages with realistic noise and characteristics:

```python
#!/usr/bin/env python3
# lidar_processor.py

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
import numpy as np

class LiDARProcessor(Node):
    def __init__(self):
        super().__init__('lidar_processor')

        # Subscribe to LiDAR data
        self.scan_sub = self.create_subscription(
            LaserScan,
            '/lidar/scan',
            self.scan_callback,
            10
        )

        # Publisher for processed data
        self.obstacle_pub = self.create_publisher(
            Float64MultiArray,
            '/obstacle_distances',
            10
        )

    def scan_callback(self, msg):
        # Process LiDAR data for obstacle detection
        ranges = np.array(msg.ranges)

        # Filter out invalid readings (inf, nan)
        valid_ranges = ranges[np.isfinite(ranges)]

        # Detect obstacles within threshold
        obstacle_threshold = 1.0  # meters
        obstacles = valid_ranges[valid_ranges < obstacle_threshold]

        # Publish obstacle information
        obstacle_msg = Float64MultiArray()
        obstacle_msg.data = [len(obstacles), np.min(valid_ranges) if len(valid_ranges) > 0 else float('inf')]
        self.obstacle_pub.publish(obstacle_msg)

def main(args=None):
    rclpy.init(args=args)
    processor = LiDARProcessor()
    rclpy.spin(processor)
    processor.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Depth Camera Simulation

### Depth Camera Configuration
Depth cameras provide RGB-D data (color + depth) for 3D scene understanding:

```xml
<!-- In SDF model -->
<sensor name="depth_camera" type="depth">
  <camera>
    <horizontal_fov>1.047</horizontal_fov>  <!-- 60 degrees -->
    <image>
      <width>640</width>
      <height>480</height>
      <format>R8G8B8</format>
    </image>
    <clip>
      <near>0.1</near>
      <far>10.0</far>
    </clip>
  </camera>
  <plugin filename="libgazebo_ros_openni_kinect.so" name="camera_plugin">
    <ros>
      <namespace>depth_camera</namespace>
      <remapping>image_raw:=/image_rect_color</remapping>
      <remapping>depth/image_raw:=/depth/image_rect_raw</remapping>
      <remapping>points:=/points</remapping>
    </ros>
    <update_rate>30</update_rate>
    <baseline>0.2</baseline>
    <distortion_k1>0.0</distortion_k1>
    <distortion_k2>0.0</distortion_k2>
    <distortion_k3>0.0</distortion_k3>
    <distortion_t1>0.0</distortion_t1>
    <distortion_t2>0.0</distortion_t2>
  </plugin>
  <always_on>true</always_on>
  <update_rate>30</update_rate>
</sensor>
```

### Depth Data Processing
Depth cameras publish multiple data streams simultaneously:

- `/depth_camera/image_rect_color` - Color image (sensor_msgs/Image)
- `/depth_camera/depth/image_rect_raw` - Depth image (sensor_msgs/Image)
- `/depth_camera/points` - Point cloud (sensor_msgs/PointCloud2)

```python
#!/usr/bin/env python3
# depth_processor.py

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image, PointCloud2
from cv_bridge import CvBridge
import cv2
import numpy as np

class DepthProcessor(Node):
    def __init__(self):
        super().__init__('depth_processor')

        self.bridge = CvBridge()

        # Subscribe to depth camera data
        self.color_sub = self.create_subscription(
            Image,
            '/depth_camera/image_rect_color',
            self.color_callback,
            10
        )

        self.depth_sub = self.create_subscription(
            Image,
            '/depth_camera/depth/image_rect_raw',
            self.depth_callback,
            10
        )

    def color_callback(self, msg):
        # Convert ROS Image to OpenCV
        cv_image = self.bridge.imgmsg_to_cv2(msg, "bgr8")

        # Process color image (e.g., object detection)
        # ... image processing code ...

    def depth_callback(self, msg):
        # Convert depth image to numpy array
        depth_image = self.bridge.imgmsg_to_cv2(msg, "32FC1")

        # Calculate distance to closest object in center region
        h, w = depth_image.shape
        center_region = depth_image[h//4:3*h//4, w//4:3*w//4]

        # Filter out invalid depth values
        valid_depths = center_region[np.isfinite(center_region)]
        if len(valid_depths) > 0:
            min_distance = np.min(valid_depths)
            self.get_logger().info(f"Closest object: {min_distance:.2f}m")

def main(args=None):
    rclpy.init(args=args)
    processor = DepthProcessor()
    rclpy.spin(processor)
    processor.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## IMU Simulation for Balance

### IMU Sensor Configuration
Inertial Measurement Units (IMUs) provide critical data for humanoid balance and orientation:

```xml
<!-- In SDF model -->
<sensor name="imu_sensor" type="imu">
  <always_on>true</always_on>
  <update_rate>100</update_rate>
  <imu>
    <angular_velocity>
      <x>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.001</stddev>
        </noise>
      </x>
      <y>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.001</stddev>
        </noise>
      </y>
      <z>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.001</stddev>
        </noise>
      </z>
    </angular_velocity>
    <linear_acceleration>
      <x>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.017</stddev>
        </noise>
      </x>
      <y>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.017</stddev>
        </noise>
      </y>
      <z>
        <noise type="gaussian">
          <mean>0.0</mean>
          <stddev>0.017</stddev>
        </noise>
      </z>
    </linear_acceleration>
  </imu>
  <plugin filename="libgazebo_ros_imu_sensor.so" name="imu_plugin">
    <ros>
      <namespace>imu</namespace>
      <remapping>~/out:=data</remapping>
    </ros>
  </plugin>
</sensor>
```

### IMU Data for Balance Control
IMU data is essential for humanoid balance control algorithms:

```python
#!/usr/bin/env python3
# balance_controller.py

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Imu
from geometry_msgs.msg import Vector3
import numpy as np
import math

class BalanceController(Node):
    def __init__(self):
        super().__init__('balance_controller')

        # Subscribe to IMU data
        self.imu_sub = self.create_subscription(
            Imu,
            '/imu/data',
            self.imu_callback,
            10
        )

        # Publisher for balance corrections
        self.correction_pub = self.create_publisher(
            Float64MultiArray,
            '/balance_corrections',
            10
        )

        # Store orientation and angular velocity
        self.orientation = None
        self.angular_velocity = None

    def imu_callback(self, msg):
        # Extract orientation (quaternion)
        quat = msg.orientation
        self.orientation = [quat.x, quat.y, quat.z, quat.w]

        # Extract angular velocity
        ang_vel = msg.angular_velocity
        self.angular_velocity = [ang_vel.x, ang_vel.y, ang_vel.z]

        # Calculate roll and pitch from quaternion
        roll, pitch, yaw = self.quaternion_to_euler(self.orientation)

        # Check balance status
        balance_threshold = 0.2  # radians
        if abs(roll) > balance_threshold or abs(pitch) > balance_threshold:
            # Generate balance correction
            correction = Float64MultiArray()
            correction.data = [roll, pitch, self.angular_velocity[0], self.angular_velocity[1]]
            self.correction_pub.publish(correction)

    def quaternion_to_euler(self, quat):
        # Convert quaternion to Euler angles
        x, y, z, w = quat

        # Roll (x-axis rotation)
        sinr_cosp = 2 * (w * x + y * z)
        cosr_cosp = 1 - 2 * (x * x + y * y)
        roll = math.atan2(sinr_cosp, cosr_cosp)

        # Pitch (y-axis rotation)
        sinp = 2 * (w * y - z * x)
        pitch = math.asin(sinp) if abs(sinp) <= 1 else math.copysign(math.pi/2, sinp)

        # Yaw (z-axis rotation)
        siny_cosp = 2 * (w * z + x * y)
        cosy_cosp = 1 - 2 * (y * y + z * z)
        yaw = math.atan2(siny_cosp, cosy_cosp)

        return roll, pitch, yaw

def main(args=None):
    rclpy.init(args=args)
    controller = BalanceController()
    rclpy.spin(controller)
    controller.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Sensor Data Integration with ROS 2

### Topic Architecture
All sensor data flows through ROS 2 topics following standard message types:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   LiDAR         │    │   Depth Camera  │    │   IMU           │
│   Sensor        │    │   Sensor        │    │   Sensor        │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          ▼                      ▼                      ▼
   /lidar/scan          /depth_camera/points      /imu/data
   sensor_msgs/         sensor_msgs/PointCloud2    sensor_msgs/
   LaserScan                                        Imu
```

### Sensor Fusion Example
Combining multiple sensor inputs for enhanced perception:

```python
#!/usr/bin/env python3
# sensor_fusion.py

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan, PointCloud2, Imu
from std_msgs.msg import Float64MultiArray
import numpy as np
from threading import Lock

class SensorFusionNode(Node):
    def __init__(self):
        super().__init__('sensor_fusion')

        # Data storage with thread safety
        self.data_lock = Lock()
        self.lidar_data = None
        self.depth_data = None
        self.imu_data = None

        # Subscribe to all sensors
        self.lidar_sub = self.create_subscription(
            LaserScan, '/lidar/scan', self.lidar_callback, 10
        )
        self.depth_sub = self.create_subscription(
            PointCloud2, '/depth_camera/points', self.depth_callback, 10
        )
        self.imu_sub = self.create_subscription(
            Imu, '/imu/data', self.imu_callback, 10
        )

        # Publisher for fused sensor data
        self.fused_pub = self.create_publisher(
            Float64MultiArray, '/fused_sensor_data', 10
        )

        # Timer for fusion processing
        self.timer = self.create_timer(0.05, self.fusion_callback)  # 20 Hz

    def lidar_callback(self, msg):
        with self.data_lock:
            self.lidar_data = msg

    def depth_callback(self, msg):
        with self.data_lock:
            self.depth_data = msg

    def imu_callback(self, msg):
        with self.data_lock:
            self.imu_data = msg

    def fusion_callback(self):
        with self.data_lock:
            if self.lidar_data and self.imu_data:
                # Simple fusion: combine key metrics
                lidar_min = min([r for r in self.lidar_data.ranges if r > 0 and r < float('inf')], default=float('inf'))
                imu_roll = self.quaternion_to_roll(self.imu_data.orientation)

                fused_data = Float64MultiArray()
                fused_data.data = [
                    lidar_min,  # Closest obstacle distance
                    imu_roll,   # Current roll angle
                    self.imu_data.angular_velocity.x,  # Roll rate
                    1.0 if lidar_min < 1.0 else 0.0   # Obstacle indicator
                ]

                self.fused_pub.publish(fused_data)

    def quaternion_to_roll(self, quat):
        # Simplified conversion to roll angle
        sinr_cosp = 2 * (quat.w * quat.x + quat.y * quat.z)
        cosr_cosp = 1 - 2 * (quat.x * quat.x + quat.y * quat.y)
        return np.arctan2(sinr_cosp, cosr_cosp)

def main(args=None):
    rclpy.init(args=args)
    fusion_node = SensorFusionNode()
    rclpy.spin(fusion_node)
    fusion_node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Quality Validation for Sensor Data

### Accuracy Testing
Validate that simulated sensor data matches expected characteristics:

- **LiDAR**: Check range accuracy, angular resolution, and noise levels
- **Depth Camera**: Verify depth accuracy, field of view, and frame rate
- **IMU**: Confirm noise characteristics and bias levels

### Performance Validation
Ensure sensor simulation maintains performance requirements:

- **Latency**: Sensor data published with &lt;100ms latency
- **Frame Rate**: Maintains specified update rates
- **Resource Usage**: Efficient CPU and memory consumption

## Summary

This chapter explored sensor simulation for humanoid robots, covering LiDAR, depth cameras, and IMU sensors. We examined configuration details, data processing techniques, and integration with ROS 2. In the next chapter, we'll connect these simulated sensors to Unity for high-fidelity visualization and human-robot interaction.