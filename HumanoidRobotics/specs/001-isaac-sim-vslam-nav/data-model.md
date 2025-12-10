# Data Model: Isaac Sim VSLAM and Navigation Module

## Overview
Data model for the Isaac Sim VSLAM and Navigation educational module, covering the entities and relationships involved in simulation, perception, and navigation workflows.

## Core Entities

### Isaac Sim Scene
- **Description**: Photorealistic environment containing objects, lighting, and materials for simulation
- **Attributes**:
  - `scene_id`: Unique identifier for the scene
  - `name`: Human-readable name of the scene
  - `description`: Detailed description of the scene's purpose and contents
  - `environment_type`: Indoor, outdoor, warehouse, etc.
  - `lighting_config`: RTX lighting configuration parameters
  - `objects`: Collection of objects in the scene with properties
  - `materials`: Material properties for realistic rendering
  - `physics_properties`: Physics simulation parameters

### Synthetic Dataset
- **Description**: Collection of generated images, depth maps, and segmentation labels for training vision models
- **Attributes**:
  - `dataset_id`: Unique identifier for the dataset
  - `name`: Human-readable name of the dataset
  - `created_date`: Date of dataset generation
  - `image_count`: Number of images in the dataset
  - `image_resolution`: Resolution of generated images (width x height)
  - `image_format`: Format of the images (e.g., PNG, JPEG)
  - `depth_maps`: Associated depth map data
  - `segmentation_labels`: Semantic segmentation labels
  - `metadata`: Additional metadata about the dataset
  - `source_scene`: Reference to the Isaac Sim Scene used for generation

### VSLAM Pipeline
- **Description**: Processing system that takes visual sensor inputs and produces pose estimates
- **Attributes**:
  - `pipeline_id`: Unique identifier for the pipeline
  - `name`: Human-readable name of the pipeline
  - `type`: Stereo, RGBD, monocular, etc.
  - `configuration`: Pipeline configuration parameters
  - `calibration_data`: Camera calibration parameters
  - `sensor_setup`: Configuration of visual sensors
  - `tracking_mode`: Feature-based, direct, or hybrid tracking
  - `optimization_method`: Backend optimization algorithm (e.g., Ceres, GTSAM)

### Robot Pose
- **Description**: Representation of the humanoid robot's position and orientation in 3D space
- **Attributes**:
  - `pose_id`: Unique identifier for the pose record
  - `timestamp`: Time when the pose was estimated
  - `position_x`: X coordinate in 3D space
  - `position_y`: Y coordinate in 3D space
  - `position_z`: Z coordinate in 3D space
  - `orientation_x`: X component of orientation quaternion
  - `orientation_y`: Y component of orientation quaternion
  - `orientation_z`: Z component of orientation quaternion
  - `orientation_w`: W component of orientation quaternion
  - `frame_id`: Reference frame for the pose
  - `source`: Source of the pose estimate (VSLAM, odometry, etc.)

### Navigation Path
- **Description**: Computed route from current location to goal that respects humanoid kinematic constraints
- **Attributes**:
  - `path_id`: Unique identifier for the path
  - `start_pose`: Starting pose for the path
  - `goal_pose`: Goal pose for the path
  - `waypoints`: Sequence of waypoints along the path
  - `cost`: Computed cost of the path
  - `constraints_respected`: Whether humanoid-specific constraints were considered
  - `planning_time`: Time taken to compute the path
  - `path_type`: Global plan, local plan, or recovery behavior
  - `validity`: Whether the path is currently valid

### Humanoid Robot Configuration
- **Description**: Configuration parameters specific to bipedal humanoid robots
- **Attributes**:
  - `config_id`: Unique identifier for the configuration
  - `robot_name`: Name of the humanoid robot model
  - `leg_length`: Length of the robot's legs
  - `step_size`: Maximum step size for walking
  - `foot_size`: Dimensions of the robot's feet
  - `balance_constraints`: Balance and stability constraints
  - `kinematic_chain`: Kinematic chain definition for the robot
  - `walking_pattern`: Default walking pattern parameters
  - `terrain_tolerance`: Tolerance for different terrain types

## Relationships

### Scene to Dataset
- **Relationship**: One-to-Many
- **Description**: One Isaac Sim Scene can be used to generate many Synthetic Datasets
- **Constraint**: Each dataset must reference exactly one scene

### Dataset to VSLAM Pipeline
- **Relationship**: Many-to-One
- **Description**: Many Synthetic Datasets can be processed by one VSLAM Pipeline
- **Constraint**: Each dataset can be processed by multiple pipelines for comparison

### VSLAM Pipeline to Robot Pose
- **Relationship**: One-to-Many
- **Description**: One VSLAM Pipeline produces many Robot Pose estimates over time
- **Constraint**: Each pose must be associated with exactly one pipeline

### Robot Pose to Navigation Path
- **Relationship**: Many-to-Many
- **Description**: Many Robot Poses can be along many Navigation Paths (as the robot moves)
- **Constraint**: At any given time, a robot pose is associated with at most one active navigation path

### Humanoid Config to Navigation Path
- **Relationship**: One-to-Many
- **Description**: One Humanoid Robot Configuration can be used for many Navigation Paths
- **Constraint**: Each navigation path must reference a humanoid configuration for constraint checking

## Data Validation Rules

### Isaac Sim Scene
- `scene_id` must be unique across all scenes
- `name` must not be empty
- `lighting_config` must be valid RTX configuration
- `environment_type` must be one of predefined values

### Synthetic Dataset
- `dataset_id` must be unique across all datasets
- `image_count` must be positive
- `image_resolution` must be valid (width and height > 0)
- `source_scene` must reference an existing scene

### VSLAM Pipeline
- `pipeline_id` must be unique across all pipelines
- `type` must be one of supported types (stereo, RGBD, monocular)
- `calibration_data` must be valid camera calibration
- `sensor_setup` must match the pipeline type

### Robot Pose
- `pose_id` must be unique across all poses
- `timestamp` must be set
- Position components must be finite numbers
- Orientation quaternion must be normalized

### Navigation Path
- `path_id` must be unique across all paths
- `start_pose` and `goal_pose` must be valid poses
- `waypoints` must form a valid path
- `cost` must be non-negative

### Humanoid Robot Configuration
- `config_id` must be unique across all configurations
- `leg_length` and `step_size` must be positive
- `balance_constraints` must be physically valid

## State Transitions

### Synthetic Dataset States
- `created`: Dataset has been generated but not yet validated
- `validated`: Dataset has passed quality checks
- `used`: Dataset has been used for training or testing
- `archived`: Dataset is no longer actively used

### VSLAM Pipeline States
- `configured`: Pipeline parameters have been set but not started
- `running`: Pipeline is actively processing sensor data
- `tracking`: Pipeline is successfully tracking pose
- `lost`: Pipeline has lost tracking
- `paused`: Pipeline is temporarily stopped

### Navigation Path States
- `planning`: Path is being computed
- `computed`: Path has been computed but not yet executed
- `executing`: Path is currently being followed
- `completed`: Path execution is complete
- `aborted`: Path execution was interrupted