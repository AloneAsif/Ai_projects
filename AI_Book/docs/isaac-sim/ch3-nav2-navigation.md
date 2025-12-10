---
sidebar_label: 'Chapter 3: Nav2 Navigation for Humanoids'
sidebar_position: 3
---

# Chapter 3: Nav2 Navigation for Humanoids

## Overview

This chapter focuses on implementing ROS 2 Navigation (Nav2) specifically for bipedal humanoid robots. Students will learn Nav2 path-planning concepts with humanoid-specific constraints, enabling them to plan and execute walking trajectories in cluttered indoor environments.

## Learning Objectives

By the end of this chapter, students will be able to:

- Configure Nav2 with humanoid-specific parameters and constraints
- Implement path planning algorithms that consider humanoid kinematics
- Set up costmap configurations for bipedal locomotion
- Create trajectory execution for walking patterns
- Implement obstacle avoidance respecting humanoid-specific constraints
- Validate navigation performance in simulation environments

## Introduction to Nav2 for Humanoids

Navigation 2 (Nav2) is the state-of-the-art navigation stack for ROS 2, but requires special configuration for humanoid robots:

- **Humanoid Kinematics**: Bipedal locomotion constraints
- **Footstep Planning**: Walking pattern generation
- **Balance Constraints**: Center of mass and stability requirements
- **Terrain Adaptation**: Different from wheeled robots

### Key Components

- **Navigation Server**: Main orchestration of navigation tasks
- **Path Planner**: Global and local path planning with humanoid constraints
- **Controller**: Trajectory execution for walking patterns
- **Costmap 2D**: Obstacle avoidance with humanoid-specific cost functions
- **Behavior Trees**: Custom navigation behaviors for humanoids

## Humanoid-Specific Nav2 Configuration

### Navigation Parameters

```yaml
# humanoid_nav2_params.yaml
bt_navigator:
  ros__parameters:
    use_sim_time: True
    global_frame: map
    robot_base_frame: base_link
    odom_topic: /odom
    bt_xml_filename: "navigate_w_replanning_and_recovery.xml"
    default_server_timeout: 20
    enable_groot_monitoring: True
    groot_zmq_publisher_port: 1666
    groot_zmq_server_port: 1667
    interrupt_on_shutdown: True

controller_server:
  ros__parameters:
    use_sim_time: True
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.001
    min_y_velocity_threshold: 0.001
    min_theta_velocity_threshold: 0.001
    progress_checker_plugin: "progress_checker"
    goal_checker_plugin: "goal_checker"
    controller_plugins: ["FollowPath"]

    # Humanoid-specific controller
    FollowPath:
      plugin: "nav2_mppi_controller::MPPIController"
      time_steps: 50
      model_dt: 0.05
      batch_size: 1000
      vx_std: 0.2
      vy_std: 0.1
      wz_std: 0.3
      vx_max: 0.3  # Reduced for humanoid stability
      vx_min: -0.1
      vy_max: 0.1
      wz_max: 0.3
      simulation_time: 2.5
      control_duration: 0.05
      replan_frequency: 0.1
      transform_tolerance: 0.1
      xy_goal_tolerance: 0.1  # Tighter tolerance for precise foot placement
      yaw_goal_tolerance: 0.1
      state_reset_threshold: 0.5
      forward_prediction: 1.0
      # Humanoid-specific cost parameters
      goal_angle_cost_mult: 3.0
      goal_pos_cost_mult: 2.0
      path_cost_mult: 1.5
      occ_cost_mult: 2.5
      smoothness_cost_mult: 1.0
      track_pos_cost_mult: 2.0
      track_orientation_cost_mult: 2.0

local_costmap:
  local_costmap:
    ros__parameters:
      update_frequency: 10.0
      publish_frequency: 5.0
      global_frame: odom
      robot_base_frame: base_link
      use_sim_time: True
      rolling_window: true
      width: 6
      height: 6
      resolution: 0.05  # Higher resolution for precise footstep planning
      robot_radius: 0.3  # Humanoid-specific radius
      plugins: ["voxel_layer", "inflation_layer"]
      inflation_layer:
        plugin: "nav2_costmap_2d::InflationLayer"
        cost_scaling_factor: 3.0  # Higher for humanoid safety
        inflation_radius: 0.5
      voxel_layer:
        plugin: "nav2_costmap_2d::VoxelLayer"
        enabled: True
        publish_voxel_map: True
        origin_z: 0.0
        z_resolution: 0.2
        z_voxels: 8
        max_obstacle_height: 2.0
        mark_threshold: 0
        observation_sources: scan
        scan:
          topic: /scan
          max_obstacle_height: 2.0
          clearing: True
          marking: True
          data_type: "LaserScan"
          raytrace_max_range: 3.0
          raytrace_min_range: 0.0
          obstacle_max_range: 2.5
          obstacle_min_range: 0.0

global_costmap:
  global_costmap:
    ros__parameters:
      update_frequency: 1.0
      publish_frequency: 0.5
      global_frame: map
      robot_base_frame: base_link
      use_sim_time: True
      robot_radius: 0.3
      resolution: 0.05
      track_unknown_space: true
      plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
      obstacle_layer:
        plugin: "nav2_costmap_2d::ObstacleLayer"
        enabled: True
        observation_sources: scan
        scan:
          topic: /scan
          max_obstacle_height: 2.0
          clearing: True
          marking: True
          data_type: "LaserScan"
          raytrace_max_range: 3.0
          raytrace_min_range: 0.0
          obstacle_max_range: 2.5
          obstacle_min_range: 0.0
      static_layer:
        plugin: "nav2_costmap_2d::StaticLayer"
        map_subscribe_transient_local: True
      inflation_layer:
        plugin: "nav2_costmap_2d::InflationLayer"
        cost_scaling_factor: 3.0
        inflation_radius: 0.5
```

## Path Planning for Humanoids

### Humanoid-Specific Path Planning

Humanoid robots have different path planning requirements compared to wheeled robots:

- **Step Constraints**: Maximum step size limitations
- **Balance Requirements**: Center of mass considerations
- **Foot Placement**: Precise foot positioning
- **Terrain Classification**: Different surfaces require different approaches

```python
# path_planner.py
import numpy as np
from nav2_msgs.action import NavigateToPose
from geometry_msgs.msg import PoseStamped, Point
from builtin_interfaces.msg import Duration
import math

class HumanoidPathPlanner:
    def __init__(self):
        # Humanoid-specific parameters
        self.max_step_size = 0.3  # meters
        self.foot_separation = 0.2  # meters
        self.balance_margin = 0.1  # safety margin for COM
        self.step_height = 0.05  # step over small obstacles

    def plan_path_with_humanoid_constraints(self, start_pose, goal_pose, costmap):
        """
        Plan path considering humanoid kinematic constraints
        """
        # 1. Plan initial path using standard A*
        initial_path = self.plan_standard_path(start_pose, goal_pose, costmap)

        # 2. Smooth path for humanoid walking
        smoothed_path = self.smooth_for_humanoid(initial_path)

        # 3. Generate footstep plan
        footsteps = self.generate_footsteps(smoothed_path)

        # 4. Validate path for humanoid constraints
        if self.validate_humanoid_path(footsteps, costmap):
            return footsteps
        else:
            # Try alternative path with more conservative constraints
            return self.plan_conservative_path(start_pose, goal_pose, costmap)

    def generate_footsteps(self, path):
        """
        Generate footstep plan from continuous path
        """
        footsteps = []

        # Start with left foot
        left_foot = True

        for i in range(len(path) - 1):
            current_pose = path[i]
            next_pose = path[i + 1]

            # Calculate direction vector
            dx = next_pose.pose.position.x - current_pose.pose.position.x
            dy = next_pose.pose.position.y - current_pose.pose.position.y
            distance = math.sqrt(dx*dx + dy*dy)

            # If distance is within step size, add footstep
            if distance <= self.max_step_size:
                foot_pose = PoseStamped()
                foot_pose.header = current_pose.header
                foot_pose.pose = current_pose.pose

                # Offset for left/right foot
                if left_foot:
                    foot_pose.pose.position.y += self.foot_separation / 2
                else:
                    foot_pose.pose.position.y -= self.foot_separation / 2

                footsteps.append(foot_pose)
                left_foot = not left_foot  # Alternate feet

        return footsteps

    def validate_humanoid_path(self, footsteps, costmap):
        """
        Validate path considering humanoid balance and step constraints
        """
        for i, footstep in enumerate(footsteps):
            # Check if footstep is in valid area
            if not self.is_valid_footstep(footstep, costmap):
                return False

            # Check balance constraints with previous footsteps
            if i > 0:
                if not self.check_balance_constraint(footsteps[i-1], footstep):
                    return False

        return True

    def is_valid_footstep(self, footstep, costmap):
        """
        Check if footstep is valid based on costmap
        """
        # Implementation would check costmap at footstep location
        # with humanoid-specific footprint
        pass

    def check_balance_constraint(self, prev_footstep, current_footstep):
        """
        Check if the step maintains humanoid balance
        """
        # Calculate center of mass position based on both feet
        # Ensure it's within support polygon
        pass
```

## Costmap Configuration for Humanoids

### Humanoid-Specific Costmap

```yaml
# humanoid_costmaps.yaml
local_costmap:
  local_costmap:
    ros__parameters:
      # Resolution optimized for footstep planning
      resolution: 0.025  # Higher resolution for precise planning
      robot_radius: 0.25  # Humanoid-specific radius

      # Plugins for humanoid navigation
      plugins: [
        "static_layer",
        "obstacle_layer",
        "voxel_layer",
        "inflation_layer"
      ]

      # Static layer for known obstacles
      static_layer:
        plugin: "nav2_costmap_2d::StaticLayer"
        enabled: True
        subscribe_to_updates: True

      # Obstacle layer for dynamic obstacles
      obstacle_layer:
        plugin: "nav2_costmap_2d::ObstacleLayer"
        enabled: True
        observation_sources: scan
        scan:
          topic: /scan
          sensor_frame: base_scan
          max_obstacle_height: 2.0
          clearing: True
          marking: True
          data_type: "LaserScan"
          raytrace_max_range: 5.0
          raytrace_min_range: 0.0
          obstacle_max_range: 4.0
          obstacle_min_range: 0.0

      # Voxel layer for 3D obstacle representation
      voxel_layer:
        plugin: "nav2_costmap_2d::VoxelLayer"
        enabled: True
        publish_voxel_map: True
        origin_z: 0.0
        z_resolution: 0.05  # Fine resolution for step height
        z_voxels: 20
        max_obstacle_height: 2.0
        mark_threshold: 0
        observation_sources: scan
        scan:
          topic: /scan
          max_obstacle_height: 2.0
          clearing: True
          marking: True
          data_type: "LaserScan"
          raytrace_max_range: 5.0
          raytrace_min_range: 0.0
          obstacle_max_range: 4.0
          obstacle_min_range: 0.0

      # Inflation layer with humanoid-specific parameters
      inflation_layer:
        plugin: "nav2_costmap_2d::InflationLayer"
        enabled: True
        cost_scaling_factor: 5.0  # Higher for humanoid safety
        inflation_radius: 0.6    # Larger for humanoid stability
        inflate_unknown: false
        inflate_around_unknown: true
```

## Walking Controller Implementation

### Trajectory Execution for Humanoids

```python
# walking_controller.py
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from nav_msgs.msg import Path
from geometry_msgs.msg import Twist
from sensor_msgs.msg import JointState
import numpy as np
from scipy import interpolate

class HumanoidWalkingController(Node):
    def __init__(self):
        super().__init__('humanoid_walking_controller')

        # Publishers for humanoid commands
        self.joint_cmd_pub = self.create_publisher(
            JointState, '/joint_commands', 10)
        self.cmd_vel_pub = self.create_publisher(
            Twist, '/cmd_vel', 10)

        # Subscribers for feedback
        self.odom_sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10)
        self.imu_sub = self.create_subscription(
            Imu, '/imu', self.imu_callback, 10)

        # Walking pattern parameters
        self.step_height = 0.05  # meters
        self.step_duration = 1.0  # seconds
        self.step_frequency = 1.0 / self.step_duration
        self.com_height = 0.8     # Center of mass height

        # Balance control
        self.balance_controller = BalanceController()

        # Timer for walking control
        self.control_timer = self.create_timer(
            0.01, self.walking_control_loop)  # 100Hz control

    def execute_walking_trajectory(self, path):
        """
        Execute walking trajectory following a path
        """
        self.get_logger().info(f'Executing walking trajectory with {len(path.poses)} waypoints')

        for i, waypoint in enumerate(path.poses):
            self.get_logger().info(f'Approaching waypoint {i+1}/{len(path.poses)}')

            # Generate walking pattern to reach waypoint
            walking_pattern = self.generate_walking_pattern(
                self.current_pose, waypoint.pose)

            # Execute the walking pattern
            self.execute_pattern(walking_pattern)

            # Wait for completion or timeout
            if not self.wait_for_waypoint_reached(waypoint.pose, timeout=30.0):
                self.get_logger().warn(f'Failed to reach waypoint {i+1}, continuing to next')
                continue

    def generate_walking_pattern(self, start_pose, target_pose):
        """
        Generate walking pattern to move from start to target pose
        """
        # Calculate required steps based on distance and max step size
        dx = target_pose.position.x - start_pose.position.x
        dy = target_pose.position.y - start_pose.position.y
        distance = np.sqrt(dx*dx + dy*dy)

        # Calculate required orientation change
        target_yaw = self.quaternion_to_yaw(target_pose.orientation)
        current_yaw = self.quaternion_to_yaw(start_pose.orientation)
        yaw_diff = target_yaw - current_yaw

        # Generate footsteps
        num_steps = int(np.ceil(distance / self.max_step_size))
        step_size = distance / num_steps if num_steps > 0 else 0

        footsteps = []
        for i in range(num_steps):
            # Calculate intermediate pose
            step_ratio = (i + 1) / num_steps
            interp_pose = Pose()
            interp_pose.position.x = start_pose.position.x + dx * step_ratio
            interp_pose.position.y = start_pose.position.y + dy * step_ratio
            interp_pose.position.z = start_pose.position.z

            # Interpolate orientation
            interp_yaw = current_yaw + yaw_diff * step_ratio
            interp_pose.orientation = self.yaw_to_quaternion(interp_yaw)

            footsteps.append(interp_pose)

        return footsteps

    def execute_pattern(self, walking_pattern):
        """
        Execute the walking pattern with balance control
        """
        for i, foot_pose in enumerate(walking_pattern):
            # Plan footstep trajectory
            foot_trajectory = self.plan_foot_trajectory(foot_pose)

            # Execute footstep with balance maintenance
            self.execute_footstep(foot_trajectory)

            # Update balance as needed
            self.balance_controller.update_balance()

    def plan_foot_trajectory(self, target_pose):
        """
        Plan smooth trajectory for foot movement
        """
        # Use 5th order polynomial for smooth movement
        # with proper lift and place motions
        pass

    def execute_footstep(self, foot_trajectory):
        """
        Execute a single footstep with proper timing
        """
        # Move foot along planned trajectory
        # Update balance based on foot placement
        # Maintain center of mass stability
        pass

class BalanceController:
    def __init__(self):
        self.com_height = 0.8  # Center of mass height
        self.com_position = np.array([0.0, 0.0, self.com_height])
        self.support_polygon = []  # Current support polygon

    def update_balance(self):
        """
        Update balance based on current foot positions
        """
        # Calculate current center of mass position
        # Ensure it's within support polygon
        # Adjust if necessary
        pass

    def calculate_zmp(self):
        """
        Calculate Zero Moment Point for balance
        """
        pass
```

## Navigation Launch Configuration

### Launch File for Humanoid Navigation

```python
# launch_navigation.py
# Implements T051: Create navigation launch file in chapter_3_navigation/nav2_configs/launch_navigation.py with humanoid parameters

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, RegisterEventHandler
from launch.conditions import IfCondition
from launch.event_handlers import OnProcessExit
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    # Launch configuration variables
    use_sim_time = LaunchConfiguration('use_sim_time')
    autostart = LaunchConfiguration('autostart')
    params_file = LaunchConfiguration('params_file')
    bt_xml_file = LaunchConfiguration('bt_xml_file')
    map_topic = LaunchConfiguration('map_topic')

    # Declare launch arguments
    declare_use_sim_time = DeclareLaunchArgument(
        'use_sim_time',
        default_value='True',
        description='Use simulation (Gazebo) clock if true'
    )

    declare_autostart = DeclareLaunchArgument(
        'autostart',
        default_value='True',
        description='Automatically startup the nav2 stack'
    )

    declare_params_file = DeclareLaunchArgument(
        'params_file',
        default_value=os.path.join(
            get_package_share_directory('humanoid_nav2_bringup'),
            'config',
            'humanoid_nav2_params.yaml'
        ),
        description='Full path to the ROS2 parameters file to use for all launched nodes'
    )

    declare_bt_xml_file = DeclareLaunchArgument(
        'bt_xml_file',
        default_value=os.path.join(
            get_package_share_directory('nav2_bt_navigator'),
            'behavior_trees',
            'navigate_w_replanning_and_recovery.xml'
        ),
        description='Full path to the behavior tree xml file to use'
    )

    # Implement FR-006: Users MUST be able to configure Nav2 path-planning for bipedal humanoid robots
    # Navigation Server
    navigation_server_node = Node(
        package='nav2_controller',
        executable='nav2_controller',
        name='navigation_server',
        parameters=[params_file],
        remappings=[('/cmd_vel', '/humanoid/cmd_vel')],
        output='screen'
    )

    # Planner Server
    planner_server_node = Node(
        package='nav2_planner',
        executable='nav2_planner',
        name='planner_server',
        parameters=[params_file],
        remappings=[
            ('/global_costmap/costmap_raw', '/global_costmap/costmap_raw'),
            ('/global_costmap/costmap', '/global_costmap/costmap'),
        ],
        output='screen'
    )

    # Recoveries Server
    recoveries_server_node = Node(
        package='nav2_recoveries',
        executable='nav2_recoveries',
        name='recoveries_server',
        parameters=[params_file],
        output='screen'
    )

    # BT Navigator
    bt_navigator_node = Node(
        package='nav2_bt_navigator',
        executable='nav2_bt_navigator',
        name='bt_navigator',
        parameters=[params_file, {'bt_xml_filename': bt_xml_file}],
        remappings=[
            ('cmd_vel', '/humanoid/cmd_vel'),
            ('global_costmap', 'global_costmap'),
            ('local_costmap', 'local_costmap'),
        ],
        output='screen'
    )

    # Lifecycle Manager
    lifecycle_manager = Node(
        package='nav2_lifecycle_manager',
        executable='lifecycle_manager',
        name='lifecycle_manager_navigation',
        output='screen',
        parameters=[{'use_sim_time': use_sim_time},
                    {'autostart': autostart},
                    {'node_names': ['navigation_server',
                                   'planner_server',
                                   'recoveries_server',
                                   'bt_navigator']}]
    )

    return LaunchDescription([
        declare_use_sim_time,
        declare_autostart,
        declare_params_file,
        declare_bt_xml_file,
        lifecycle_manager,
        navigation_server_node,
        planner_server_node,
        recoveries_server_node,
        bt_navigator_node,
    ])


# Humanoid-specific navigation configuration
# Implements FR-007: System MUST compute valid navigation paths that respect humanoid-specific constraints

def generate_humanoid_navigation_launch():
    """
    Alternative launch configuration specifically for humanoid robots
    """
    # Launch configuration variables
    use_sim_time = LaunchConfiguration('use_sim_time')
    autostart = LaunchConfiguration('autostart')
    params_file = LaunchConfiguration('params_file')
    bt_xml_file = LaunchConfiguration('bt_xml_file')

    # Declare launch arguments
    declare_use_sim_time = DeclareLaunchArgument(
        'use_sim_time',
        default_value='True',
        description='Use simulation (Isaac Sim) clock if true'
    )

    declare_autostart = DeclareLaunchArgument(
        'autostart',
        default_value='True',
        description='Automatically startup the nav2 stack'
    )

    declare_params_file = DeclareLaunchArgument(
        'params_file',
        default_value=os.path.join(
            get_package_share_directory('humanoid_nav2_bringup'),
            'config',
            'humanoid_nav2_params.yaml'
        ),
        description='Full path to the ROS2 parameters file to use for all launched nodes'
    )

    declare_bt_xml_file = DeclareLaunchArgument(
        'bt_xml_file',
        default_value=os.path.join(
            get_package_share_directory('humanoid_nav2_bringup'),
            'behavior_trees',
            'humanoid_navigate_w_replanning_and_recovery.xml'
        ),
        description='Full path to the humanoid-specific behavior tree xml file to use'
    )

    # Humanoid-specific navigation server with walking controller
    humanoid_navigator = Node(
        package='nav2_bt_navigator',
        executable='bt_navigator',
        name='humanoid_bt_navigator',
        parameters=[params_file, {'bt_xml_filename': bt_xml_file}],
        remappings=[
            ('cmd_vel', '/humanoid/cmd_vel'),
            ('global_costmap', 'global_costmap'),
            ('local_costmap', 'local_costmap'),
            ('navigate_to_pose', 'humanoid_navigate_to_pose'),
        ],
        output='screen'
    )

    # Humanoid-specific controller server
    humanoid_controller = Node(
        package='nav2_regulated_pure_pursuit_controller',
        executable='regulated_pure_pursuit_controller',
        name='humanoid_controller_server',
        parameters=[params_file],
        remappings=[
            ('cmd_vel', '/humanoid/cmd_vel'),
            ('odom', '/humanoid/odom'),
            ('cmd_vel_output', '/humanoid/walking_controller/cmd_vel'),
        ],
        output='screen'
    )

    # Footstep planner for bipedal locomotion
    footstep_planner = Node(
        package='humanoid_nav2_planner',
        executable='footstep_planner',
        name='footstep_planner',
        parameters=[params_file],
        remappings=[
            ('/global_plan', '/humanoid/global_plan'),
            ('/footstep_plan', '/humanoid/footstep_plan'),
        ],
        output='screen'
    )

    # Balance controller for humanoid stability
    balance_controller = Node(
        package='humanoid_balance_controller',
        executable='balance_controller',
        name='balance_controller',
        parameters=[params_file],
        remappings=[
            ('/imu', '/humanoid/imu'),
            ('/center_of_mass', '/humanoid/center_of_mass'),
            ('/balance_cmd', '/humanoid/balance_cmd'),
        ],
        output='screen'
    )

    # Lifecycle Manager for humanoid navigation
    humanoid_lifecycle_manager = Node(
        package='nav2_lifecycle_manager',
        executable='lifecycle_manager',
        name='humanoid_lifecycle_manager',
        output='screen',
        parameters=[{'use_sim_time': use_sim_time},
                    {'autostart': autostart},
                    {'node_names': ['humanoid_bt_navigator',
                                   'humanoid_controller_server',
                                   'footstep_planner',
                                   'balance_controller']}]
    )

    return LaunchDescription([
        declare_use_sim_time,
        declare_autostart,
        declare_params_file,
        declare_bt_xml_file,
        humanoid_lifecycle_manager,
        humanoid_navigator,
        humanoid_controller,
        footstep_planner,
        balance_controller,
    ])
```

### Humanoid Navigation Configuration Files

Create the humanoid-specific configuration file at `chapter_3_navigation/configs/humanoid_configs.yaml`:

```yaml
# Humanoid robot configuration for navigation
# Implements FR-007: System MUST compute valid navigation paths that respect humanoid-specific constraints

# Robot-specific parameters
humanoid_robot:
  # Physical dimensions
  base_width: 0.3      # Width of the robot base in meters
  base_length: 0.3     # Length of the robot base in meters
  base_height: 0.8     # Height of the robot base in meters

  # Kinematic constraints
  max_step_size: 0.3   # Maximum step size in meters
  min_step_size: 0.05  # Minimum step size in meters
  step_height: 0.05    # Maximum step height in meters
  step_frequency: 0.5  # Steps per second

  # Balance constraints
  com_height: 0.8      # Center of mass height in meters
  balance_margin: 0.1  # Safety margin for balance in meters
  max_tilt_angle: 10.0 # Maximum tilt angle in degrees

# Footstep planning parameters
footstep_planning:
  # Step constraints
  max_step_x: 0.3      # Maximum forward step in meters
  max_step_y: 0.15     # Maximum lateral step in meters
  max_step_theta: 0.3  # Maximum turning step in radians

  # Planning parameters
  step_resolution: 0.05 # Step resolution in meters
  angle_resolution: 0.1 # Angle resolution in radians
  max_planning_time: 5.0 # Maximum planning time in seconds

# Walking controller parameters
walking_controller:
  # Gait parameters
  step_duration: 1.0    # Duration of each step in seconds
  lift_height: 0.05     # Foot lift height in meters
  swing_period: 0.3     # Swing phase duration in seconds
  stance_period: 0.7    # Stance phase duration in seconds

  # Control parameters
  linear_vel_max: 0.3   # Maximum linear velocity in m/s
  linear_vel_min: 0.05  # Minimum linear velocity in m/s
  angular_vel_max: 0.3  # Maximum angular velocity in rad/s
  acceleration_max: 0.1 # Maximum acceleration in m/s^2

# Balance control parameters
balance_control:
  # Center of mass control
  com_control_gain: 10.0
  com_tolerance: 0.05   # Center of mass tolerance in meters

  # ZMP (Zero Moment Point) control
  zmp_tolerance: 0.05   # ZMP tolerance in meters
  zmp_control_gain: 5.0

# Navigation-specific parameters
navigation:
  # Goal tolerance
  xy_goal_tolerance: 0.1  # Position tolerance in meters
  yaw_goal_tolerance: 0.1 # Orientation tolerance in radians

  # Path following
  path_tolerance: 0.1     # Path following tolerance in meters
  velocity_scaling_factor: 0.8  # Scale velocity based on stability

# Implement SC-006: Humanoid navigation paths computed by Nav2 respect kinematic constraints and avoid obstacles in 95% of test scenarios
```

### Navigation Goal Service Implementation

Create the navigation service at `chapter_3_navigation/path_planning/navigation_service.py`:

```python
#!/usr/bin/env python3
"""
Humanoid Navigation Goal Service
Implements the navigation goal service for humanoid robots with constraint-aware path planning
"""

import rclpy
from rclpy.action import ActionServer, CancelResponse, GoalResponse
from rclpy.node import Node
from rclpy.qos import QoSProfile
from nav2_msgs.action import NavigateToPose
from geometry_msgs.msg import PoseStamped
from nav_msgs.msg import Path
from sensor_msgs.msg import LaserScan
from tf2_ros import TransformException
from tf2_ros.buffer import Buffer
from tf2_ros.transform_listener import TransformListener
import numpy as np
from scipy.spatial.transform import Rotation as R
import math

class HumanoidNavigationService(Node):
    """
    Implements FR-006: Users MUST be able to configure Nav2 path-planning for bipedal humanoid robots
    Implements FR-007: System MUST compute valid navigation paths that respect humanoid-specific constraints
    """

    def __init__(self):
        super().__init__('humanoid_navigation_service')

        # Initialize action server
        self._action_server = ActionServer(
            self,
            NavigateToPose,
            'humanoid_navigate_to_pose',
            execute_callback=self.execute_callback,
            goal_callback=self.goal_callback,
            cancel_callback=self.cancel_callback
        )

        # TF buffer and listener for coordinate transforms
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)

        # Publishers and subscribers
        self.path_publisher = self.create_publisher(Path, 'humanoid/global_plan', 1)
        self.laser_sub = self.create_subscription(
            LaserScan, 'scan', self.laser_callback, 10
        )

        # Humanoid-specific parameters
        self.max_step_size = 0.3  # meters
        self.step_frequency = 0.5  # steps per second
        self.balance_margin = 0.1  # meters
        self.com_height = 0.8  # center of mass height in meters

        # Current navigation state
        self.current_goal = None
        self.is_navigating = False
        self.navigation_cancelled = False

        self.get_logger().info('Humanoid Navigation Service initialized')

    def goal_callback(self, goal_request):
        """Check if the goal request should be accepted or rejected"""
        # Check if humanoid can physically reach the goal given constraints
        goal_pose = goal_request.pose.pose
        current_pose = self.get_current_pose()

        if current_pose is None:
            self.get_logger().warn('Cannot get current pose, rejecting goal')
            return GoalResponse.REJECT

        # Check if goal is reachable considering humanoid constraints
        distance = self.calculate_distance(current_pose, goal_pose)
        if distance > 100.0:  # Arbitrary maximum distance
            self.get_logger().warn('Goal too far, rejecting')
            return GoalResponse.REJECT

        # Implement SC-006: Check if path respects humanoid kinematic constraints
        if self.is_path_feasible(current_pose, goal_pose):
            self.get_logger().info('Goal accepted')
            return GoalResponse.ACCEPT
        else:
            self.get_logger().warn('Path not feasible for humanoid, rejecting goal')
            return GoalResponse.REJECT

    def cancel_callback(self, goal_handle):
        """Accept or reject a client request to cancel the navigation goal"""
        self.get_logger().info('Received cancel request')
        return CancelResponse.ACCEPT

    def execute_callback(self, goal_handle):
        """Execute the navigation goal"""
        self.get_logger().info('Executing navigation goal')

        goal_pose = goal_pose = goal_handle.request.pose
        feedback_msg = NavigateToPose.Feedback()
        result = NavigateToPose.Result()

        # Set current goal
        self.current_goal = goal_pose
        self.is_navigating = True
        self.navigation_cancelled = False

        try:
            # Plan path considering humanoid constraints
            path = self.plan_humanoid_path(goal_pose)

            if path is None or len(path.poses) == 0:
                self.get_logger().error('Failed to plan path')
                goal_handle.abort()
                result.result = result.FAILURE
                return result

            # Publish the planned path
            self.path_publisher.publish(path)

            # Execute the path with humanoid-aware controller
            success = self.execute_humanoid_path(path, goal_pose, goal_handle, feedback_msg)

            if success:
                self.get_logger().info('Navigation succeeded')
                goal_handle.succeed()
                result.result = result.SUCCESS
            else:
                self.get_logger().info('Navigation failed')
                goal_handle.abort()
                result.result = result.FAILURE

        except Exception as e:
            self.get_logger().error(f'Navigation execution failed: {str(e)}')
            goal_handle.abort()
            result.result = result.FAILURE

        self.is_navigating = False
        self.current_goal = None

        return result

    def plan_humanoid_path(self, goal_pose):
        """
        Plan path considering humanoid-specific constraints
        Implements FR-007: Compute valid navigation paths that respect humanoid-specific constraints
        """
        try:
            # Get current pose
            current_pose = self.get_current_pose()
            if current_pose is None:
                return None

            # Plan initial path using standard path planner
            initial_path = self.plan_standard_path(current_pose, goal_pose)
            if initial_path is None:
                return None

            # Refine path for humanoid constraints
            humanoid_path = self.refine_path_for_humanoid(initial_path)

            # Validate path for humanoid feasibility
            if self.validate_humanoid_path(humanoid_path):
                return humanoid_path
            else:
                # Try alternative path planning with more conservative constraints
                return self.plan_conservative_humanoid_path(current_pose, goal_pose)

        except Exception as e:
            self.get_logger().error(f'Path planning failed: {str(e)}')
            return None

    def refine_path_for_humanoid(self, path):
        """
        Refine a path to make it suitable for humanoid navigation
        """
        refined_path = Path()
        refined_path.header = path.header

        if len(path.poses) < 2:
            return path

        # Ensure path points are within max step size for humanoid
        refined_poses = []
        for i in range(len(path.poses) - 1):
            current_pose = path.poses[i]
            next_pose = path.poses[i + 1]

            # Calculate distance between consecutive poses
            dx = next_pose.pose.position.x - current_pose.pose.position.x
            dy = next_pose.pose.position.y - current_pose.pose.position.y
            distance = math.sqrt(dx*dx + dy*dy)

            # If distance is too large, interpolate intermediate steps
            if distance > self.max_step_size:
                # Calculate number of intermediate steps needed
                num_steps = int(math.ceil(distance / self.max_step_size))

                for j in range(num_steps):
                    ratio = j / num_steps
                    interp_pose = PoseStamped()
                    interp_pose.header = path.header
                    interp_pose.pose.position.x = current_pose.pose.position.x + dx * ratio
                    interp_pose.pose.position.y = current_pose.pose.position.y + dy * ratio
                    interp_pose.pose.position.z = current_pose.pose.position.z

                    # Interpolate orientation
                    current_q = current_pose.pose.orientation
                    next_q = next_pose.pose.orientation
                    interp_pose.pose.orientation = self.interpolate_quaternion(
                        current_q, next_q, ratio
                    )

                    refined_poses.append(interp_pose)
            else:
                refined_poses.append(current_pose)

        # Add the final pose
        refined_poses.append(path.poses[-1])

        refined_path.poses = refined_poses
        return refined_path

    def validate_humanoid_path(self, path):
        """
        Validate that a path is feasible for humanoid navigation
        """
        if len(path.poses) < 2:
            return True

        for i in range(len(path.poses) - 1):
            current_pose = path.poses[i].pose
            next_pose = path.poses[i + 1].pose

            # Check step size constraint
            dx = next_pose.position.x - current_pose.position.x
            dy = next_pose.position.y - current_pose.position.y
            step_distance = math.sqrt(dx*dx + dy*dy)

            if step_distance > self.max_step_size:
                return False

            # Check for obstacles in path
            if self.is_path_obstructed(current_pose, next_pose):
                return False

        return True

    def execute_humanoid_path(self, path, goal_pose, goal_handle, feedback_msg):
        """
        Execute the path with humanoid-aware navigation
        Implements FR-008: System MUST provide reproducible examples for ROS 2 integration
        """
        self.get_logger().info(f'Executing path with {len(path.poses)} waypoints')

        for i, waypoint in enumerate(path.poses):
            if self.navigation_cancelled:
                return False

            # Calculate distance to waypoint
            current_pose = self.get_current_pose()
            if current_pose is None:
                continue

            dist_to_waypoint = self.calculate_distance(current_pose, waypoint.pose)

            # Check if close enough to this waypoint
            if dist_to_waypoint <= 0.1:  # 10cm tolerance
                continue

            # Navigate to this waypoint with humanoid-aware controller
            success = self.navigate_to_waypoint(waypoint, goal_handle, feedback_msg)

            if not success:
                self.get_logger().warn(f'Failed to reach waypoint {i+1}')
                return False

            # Update feedback
            feedback_msg.current_pose = current_pose
            feedback_msg.distance_remaining = self.calculate_distance(
                self.get_current_pose(), goal_pose.pose
            )
            goal_handle.publish_feedback(feedback_msg)

        # Check if reached final goal
        current_pose = self.get_current_pose()
        if current_pose:
            final_distance = self.calculate_distance(current_pose, goal_pose.pose)
            if final_distance <= 0.1:  # Goal tolerance
                return True

        return False

    def navigate_to_waypoint(self, waypoint, goal_handle, feedback_msg):
        """
        Navigate to a specific waypoint using humanoid-aware control
        """
        # This would interface with the humanoid walking controller
        # For simulation, we'll just check if the robot can reach the waypoint
        timeout = 30.0  # seconds
        start_time = self.get_clock().now()

        while rclpy.ok():
            if self.navigation_cancelled:
                return False

            current_pose = self.get_current_pose()
            if current_pose is None:
                continue

            distance = self.calculate_distance(current_pose, waypoint.pose)

            if distance <= 0.1:  # Waypoint reached
                return True

            # Check for timeout
            elapsed = (self.get_clock().now() - start_time).nanoseconds / 1e9
            if elapsed > timeout:
                self.get_logger().warn('Navigation timeout')
                return False

            # Update feedback periodically
            if int(elapsed) % 5 == 0:  # Every 5 seconds
                feedback_msg.current_pose = current_pose
                goal_handle.publish_feedback(feedback_msg)

            # Sleep briefly to allow other processes
            self._rate.sleep()

        return False

    def get_current_pose(self):
        """Get the current robot pose from TF"""
        try:
            transform = self.tf_buffer.lookup_transform(
                'map', 'base_link', rclpy.time.Time()
            )

            pose = PoseStamped()
            pose.pose.position.x = transform.transform.translation.x
            pose.pose.position.y = transform.transform.translation.y
            pose.pose.position.z = transform.transform.translation.z
            pose.pose.orientation = transform.transform.rotation

            return pose
        except TransformException as ex:
            self.get_logger().warn(f'Could not get current pose: {ex}')
            return None

    def calculate_distance(self, pose1, pose2):
        """Calculate Euclidean distance between two poses"""
        dx = pose2.position.x - pose1.position.x
        dy = pose2.position.y - pose1.position.y
        dz = pose2.position.z - pose1.position.z
        return math.sqrt(dx*dx + dy*dy + dz*dz)

    def interpolate_quaternion(self, q1, q2, ratio):
        """Interpolate between two quaternions"""
        # Simple linear interpolation (slerp would be better but more complex)
        import numpy as np

        # Convert to numpy arrays
        a = np.array([q1.x, q1.y, q1.z, q1.w])
        b = np.array([q2.x, q2.y, q2.z, q2.w])

        # Linear interpolation
        result = a * (1.0 - ratio) + b * ratio
        result_norm = np.linalg.norm(result)

        if result_norm > 0:
            result = result / result_norm
        else:
            result = np.array([0, 0, 0, 1])  # Identity quaternion

        from geometry_msgs.msg import Quaternion
        q_result = Quaternion()
        q_result.x = float(result[0])
        q_result.y = float(result[1])
        q_result.z = float(result[2])
        q_result.w = float(result[3])

        return q_result

    def is_path_obstructed(self, pose1, pose2):
        """Check if the path between two poses is obstructed"""
        # This would check laser scan data or costmap for obstacles
        # Simplified implementation for now
        return False

    def plan_standard_path(self, start_pose, goal_pose):
        """Plan a standard path (would interface with Nav2 planner)"""
        # This would normally call the Nav2 planner service
        # For this example, we'll create a simple straight-line path
        path = Path()
        path.header.frame_id = 'map'

        # Create a simple path from start to goal
        steps = 20
        for i in range(steps + 1):
            ratio = i / steps
            step_pose = PoseStamped()
            step_pose.header.frame_id = 'map'

            step_pose.pose.position.x = start_pose.pose.position.x + \
                (goal_pose.pose.position.x - start_pose.pose.position.x) * ratio
            step_pose.pose.position.y = start_pose.pose.position.y + \
                (goal_pose.pose.position.y - start_pose.pose.position.y) * ratio
            step_pose.pose.position.z = start_pose.pose.position.z + \
                (goal_pose.pose.position.z - start_pose.pose.position.z) * ratio

            # For orientation, use the goal orientation
            step_pose.pose.orientation = goal_pose.pose.orientation

            path.poses.append(step_pose)

        return path

    def plan_conservative_humanoid_path(self, start_pose, goal_pose):
        """Plan a conservative path for humanoid navigation"""
        # Plan path with extra safety margins for humanoid
        return self.plan_standard_path(start_pose, goal_pose)

    def laser_callback(self, msg):
        """Handle laser scan data for obstacle detection"""
        # Store laser data for path validation
        self.last_laser_scan = msg

def main(args=None):
    rclpy.init(args=args)

    navigation_service = HumanoidNavigationService()

    try:
        rclpy.spin(navigation_service)
    except KeyboardInterrupt:
        pass
    finally:
        navigation_service.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Practical Exercise: Humanoid Navigation Setup

### Step 1: Configure Nav2 for Humanoid

1. Set up humanoid-specific navigation parameters
2. Configure costmap with appropriate resolution and inflation
3. Test parameter loading with navigation server

### Step 2: Implement Path Planning

1. Create path planner with humanoid constraints
2. Generate footstep plans from continuous paths
3. Validate paths for balance and step constraints

### Step 3: Execute Navigation

1. Launch navigation stack in Isaac Sim environment
2. Send navigation goals to the robot
3. Observe and analyze navigation behavior

## Obstacle Avoidance with Humanoid Constraints

### Humanoid-Aware Obstacle Avoidance

```python
# obstacle_avoidance.py
class HumanoidObstacleAvoidance:
    def __init__(self):
        self.min_step_distance = 0.1  # Minimum distance for foot placement
        self.max_step_up = 0.1        # Maximum step-up height
        self.max_step_down = 0.05     # Maximum step-down height
        self.com_stability_margin = 0.05  # Stability safety margin

    def find_safe_path_around_obstacle(self, obstacle_pos, obstacle_size, target_pos):
        """
        Find safe path around obstacle considering humanoid constraints
        """
        # Calculate safe distance based on humanoid size
        safe_distance = max(
            self.robot_radius + obstacle_size.x/2,
            self.robot_radius + obstacle_size.y/2
        ) + self.min_step_distance

        # Generate multiple path options around the obstacle
        path_options = self.generate_path_options(
            obstacle_pos, obstacle_size, safe_distance, target_pos)

        # Evaluate each option for humanoid feasibility
        best_path = None
        best_score = float('inf')

        for path in path_options:
            score = self.evaluate_humanoid_path_feasibility(path)
            if score < best_score:
                best_score = score
                best_path = path

        return best_path

    def evaluate_humanoid_path_feasibility(self, path):
        """
        Evaluate path feasibility for humanoid navigation
        """
        score = 0.0

        for i in range(len(path) - 1):
            step = self.calculate_step(path[i], path[i+1])

            # Check step size constraint
            if step.distance > self.max_step_size:
                score += 1000  # Large penalty for impossible steps

            # Check step height constraint
            if abs(step.height_change) > self.max_step_up:
                score += 500   # Penalty for height violations

            # Check balance during step
            if not self.is_balance_maintained_during_step(step):
                score += 1000  # Large penalty for unstable steps

        return score

    def generate_path_options(self, obstacle_pos, obstacle_size, safe_distance, target_pos):
        """
        Generate multiple path options around an obstacle
        """
        # Generate paths around left, right, top, bottom of obstacle
        # considering humanoid constraints
        pass
```

## Validation and Testing

### Navigation Performance Metrics

```python
# path_execution_monitor.py
class PathExecutionMonitor:
    def __init__(self):
        self.path_computed_time = None
        self.path_execution_start = None
        self.path_execution_end = None
        self.successful_waypoints = 0
        self.total_waypoints = 0
        self.path_length = 0.0
        self.executed_path_length = 0.0

    def validate_navigation_performance(self):
        """
        Validate navigation performance against requirements
        """
        metrics = {
            'path_computation_time': self.get_path_computation_time(),
            'path_execution_time': self.get_path_execution_time(),
            'success_rate': self.get_success_rate(),
            'path_efficiency': self.get_path_efficiency(),
            'deviation_from_plan': self.get_deviation_from_plan(),
            'balance_stability': self.get_balance_stability(),
        }

        # Check against requirements
        requirements = {
            'path_computation_time': 2.0,  # seconds
            'min_success_rate': 0.95,      # 95%
            'max_deviation': 0.1,          # meters
        }

        return self.evaluate_against_requirements(metrics, requirements)

    def get_path_computation_time(self):
        """Get time taken to compute navigation path"""
        if self.path_computed_time and self.path_execution_start:
            return (self.path_execution_start - self.path_computed_time).nanoseconds / 1e9
        return float('inf')

    def get_path_execution_time(self):
        """Get time taken to execute navigation path"""
        if self.path_execution_start and self.path_execution_end:
            return (self.path_execution_end - self.path_execution_start).nanoseconds / 1e9
        return float('inf')

    def get_success_rate(self):
        """Get navigation success rate"""
        if self.total_waypoints > 0:
            return self.successful_waypoints / self.total_waypoints
        return 0.0
```

## Troubleshooting

### Common Issues

- **Balance Loss**: Center of mass instability during navigation
- **Step Size Violations**: Steps too large for humanoid kinematics
- **Path Planning Failures**: Inability to find valid paths
- **Performance Issues**: Slow path computation or execution

### Solutions

- **Balance Control**: Implement proper balance control algorithms
- **Path Smoothing**: Smooth paths for feasible step sequences
- **Parameter Tuning**: Adjust costmap and controller parameters
- **Footstep Planning**: Use proper footstep planning algorithms

## Summary

This chapter covered the implementation of Nav2 navigation specifically for humanoid robots. Students learned to configure Nav2 with humanoid-specific parameters, implement path planning algorithms considering bipedal constraints, and execute walking trajectories while maintaining balance. The module provides a complete pipeline from Isaac Sim simulation through VSLAM perception to humanoid navigation execution.

## Further Reading

- [Navigation 2 (Nav2) Documentation](https://navigation.ros.org/)
- [Humanoid Robot Navigation](https://humanoid-navigation.readthedocs.io/)
- [ROS 2 Control for Humanoids](https://control.ros.org/)
- [Bipedal Locomotion Control](https://bipedal-control.readthedocs.io/)