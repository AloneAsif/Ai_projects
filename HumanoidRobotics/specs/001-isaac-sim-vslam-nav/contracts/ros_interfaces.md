# Isaac ROS and Nav2 Interface Contracts

## Overview
This document defines the ROS 2 interfaces and API contracts for the Isaac Sim VSLAM and Navigation educational module. These interfaces enable communication between Isaac Sim, Isaac ROS perception pipeline, and Nav2 navigation stack.

## Isaac ROS Perception Interfaces

### Visual SLAM Services and Topics

#### Pose Estimation Topics
```yaml
# Published pose from VSLAM pipeline
topic: /visual_slam/tracking/pose
type: geometry_msgs/PoseStamped
description: Robot pose estimate from visual SLAM pipeline
message_format:
  header:
    stamp:
      sec: int32
      nanosec: uint32
    frame_id: string
  pose:
    position:
      x: float64
      y: float64
      z: float64
    orientation:
      x: float64
      y: float64
      z: float64
      w: float64
```

#### Visual Odometry Topics
```yaml
# Stereo camera input topics
topics:
  - name: /stereo_camera/left/image_rect_color
    type: sensor_msgs/Image
    description: Left camera image for stereo processing
  - name: /stereo_camera/right/image_rect_color
    type: sensor_msgs/Image
    description: Right camera image for stereo processing
  - name: /stereo_camera/left/camera_info
    type: sensor_msgs/CameraInfo
    description: Calibration information for left camera
  - name: /stereo_camera/right/camera_info
    type: sensor_msgs/CameraInfo
    description: Calibration information for right camera
```

#### Feature Tracking Topics
```yaml
# Feature tracking output
topic: /visual_slam/tracking/feature_tracks
type: vision_msgs/TrackedPointArray
description: Tracked visual features for SLAM
message_format:
  header:
    stamp:
      sec: int32
      nanosec: uint32
    frame_id: string
  tracks:
    - point:
        x: float64
        y: float64
        z: float64
      tracking_error: float64
      visibility: uint8
```

## Nav2 Navigation Interfaces

### Navigation Actions

#### Navigate To Pose Action
```yaml
action: /navigate_to_pose
type: nav2_msgs/action/NavigateToPose
description: Navigate the humanoid robot to a specific pose
goal_format:
  pose:
    header:
      stamp:
        sec: int32
        nanosec: uint32
      frame_id: string
    pose:
      position:
        x: float64
        y: float64
        z: float64
      orientation:
        x: float64
        y: float64
        z: float64
        w: float64
  behavior_tree: string  # Optional behavior tree to use
result_format:
  result_code: int8
  # SUCCESS = 1, FAILURE = 0, CANCELED = -1
```

#### Compute Path to Pose Service
```yaml
service: /compute_path_to_pose
type: nav2_msgs/srv/ComputePathToPose
description: Compute a navigation path to a specific pose
request_format:
  goal:
    header:
      stamp:
        sec: int32
        nanosec: uint32
      frame_id: string
    pose:
      position:
        x: float64
        y: float64
        z: float64
      orientation:
        x: float64
        y: float64
        z: float64
        w: float64
  planner_id: string  # ID of the planner to use
  use_start: bool     # Whether to use provided start pose
  start:
    header:
      stamp:
        sec: int32
        nanosec: uint32
      frame_id: string
    pose:
      position:
        x: float64
        y: float64
        z: float64
      orientation:
        x: float64
        y: float64
        z: float64
        w: float64
response_format:
  path:
    header:
      stamp:
        sec: int32
        nanosec: uint32
      frame_id: string
    poses:
      - header:
          stamp:
            sec: int32
            nanosec: uint32
          frame_id: string
        pose:
          position:
            x: float64
            y: float64
            z: float64
          orientation:
            x: float64
            y: float64
            z: float64
            w: float64
  error_code: int8
```

### Costmap and Map Services

#### Map Service
```yaml
service: /map_server/map
type: nav_msgs/srv/GetMap
description: Retrieve the current map for navigation
request_format: {}
response_format:
  map:
    header:
      stamp:
        sec: int32
        nanosec: uint32
      frame_id: string
    map_load_time:
      sec: int32
      nanosec: uint32
    resolution: float32
    width: uint32
    height: uint32
    origin:
      position:
        x: float64
        y: float64
        z: float64
      orientation:
        x: float64
        y: float64
        z: float64
        w: float64
    data: int8[]  # Array of occupancy values
```

#### Costmap Topics
```yaml
# Local and global costmap topics
topics:
  - name: /local_costmap/costmap
    type: nav_msgs/OccupancyGrid
    description: Local costmap for obstacle avoidance
  - name: /global_costmap/costmap
    type: nav_msgs/OccupancyGrid
    description: Global costmap for path planning
  - name: /local_costmap/costmap_updates
    type: map_msgs/OccupancyGridUpdate
    description: Incremental updates to local costmap
  - name: /global_costmap/costmap_updates
    type: map_msgs/OccupancyGridUpdate
    description: Incremental updates to global costmap
```

## Isaac Sim Integration Interfaces

### Simulation Control Services
```yaml
# Services for controlling Isaac Sim simulation
services:
  - name: /isaac_sim/reset_simulation
    type: std_srvs/srv/Empty
    description: Reset the simulation to initial state
  - name: /isaac_sim/pause_simulation
    type: std_srvs/srv/Empty
    description: Pause the simulation
  - name: /isaac_sim/unpause_simulation
    type: std_srvs/srv/Empty
    description: Resume the simulation
  - name: /isaac_sim/set_simulation_speed
    type: example_interfaces/srv/Float64
    description: Set simulation speed multiplier
```

### Sensor Data Topics from Simulation
```yaml
# Various sensor data topics published by Isaac Sim
topics:
  - name: /camera/rgb/image_raw
    type: sensor_msgs/Image
    description: RGB camera image from simulated camera
  - name: /camera/depth/image_raw
    type: sensor_msgs/Image
    description: Depth image from simulated depth sensor
  - name: /camera/segmentation/image_raw
    type: sensor_msgs/Image
    description: Semantic segmentation from simulated camera
  - name: /imu/data
    type: sensor_msgs/Imu
    description: Inertial measurement unit data
  - name: /odom
    type: nav_msgs/Odometry
    description: Ground truth odometry from simulation
```

## Humanoid-Specific Navigation Parameters

### Navigation Parameters (YAML format)
```yaml
# Humanoid-specific navigation parameters
local_costmap:
  local_costmap:
    ros__parameters:
      update_frequency: 5.0
      publish_frequency: 2.0
      global_frame: odom
      robot_base_frame: base_link
      use_sim_time: true
      resolution: 0.05
      footprint: "[[-0.3, -0.2], [-0.3, 0.2], [0.3, 0.2], [0.3, -0.2]]"  # Humanoid footprint
      plugins: ["voxel_layer", "inflation_layer"]
      inflation_layer:
        inflation_radius: 0.5  # Humanoid-specific inflation for bipedal movement
        cost_scaling_factor: 3.0

global_costmap:
  global_costmap:
    ros__parameters:
      update_frequency: 1.0
      publish_frequency: 1.0
      global_frame: map
      robot_base_frame: base_link
      use_sim_time: true
      resolution: 0.05
      plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
      inflation_layer:
        inflation_radius: 0.6  # Humanoid-specific inflation
        cost_scaling_factor: 2.5

controller_server:
  ros__parameters:
    use_sim_time: true
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.05
    min_y_velocity_threshold: 0.05
    min_theta_velocity_threshold: 0.05
    progress_checker_plugin: "progress_checker"
    goal_checker_plugin: "goal_checker"
    controller_plugins: ["FollowPath"]

    # Humanoid-specific controller
    FollowPath:
      plugin: "nav2_mppi_controller::MPPIController"
      time_steps: 50
      model_dt: 0.05
      no_samples: 100
      motion_model: "DiffDrive"
      x_std: 0.5
      y_std: 0.05
      theta_std: 0.05
      control_duration: 0.1
      frequency: 20.0
      velocity_scaling_window: 1.0
```

## Data Export Interfaces

### Synthetic Dataset Export Service
```yaml
# Service for exporting synthetic datasets from Isaac Sim
service: /synthetic_data/export_dataset
type: example_interfaces/srv/Trigger
description: Export current simulation data as synthetic dataset
request_format: {}
response_format:
  success: bool
  message: string
  dataset_path: string  # Path where dataset was saved
```

### Dataset Information Topic
```yaml
# Topic for dataset metadata
topic: /synthetic_data/dataset_info
type: std_msgs/String
description: JSON string with dataset metadata
message_format:
  data: string  # JSON string containing dataset information
  # Example: {"id": "dataset_001", "images": 100, "format": "RGBD", "labels": ["person", "chair", "table"]}
```