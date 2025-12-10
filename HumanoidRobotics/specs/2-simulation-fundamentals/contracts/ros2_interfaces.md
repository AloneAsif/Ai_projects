# ROS 2 Service and Message Definitions for Simulation Fundamentals

## Gazebo Services

### /spawn_entity
**Service Type**: `gazebo_msgs/srv/SpawnEntity`

**Request**:
- `name`: Name of the entity to spawn (string)
- `xml`: URDF/XML description of the entity (string)
- `robot_namespace`: Robot namespace (string, optional)
- `initial_pose`: Initial pose of the entity (geometry_msgs/Pose)
  - `position`: 3D position (x, y, z) (float[3])
  - `orientation`: 4D orientation (x, y, z, w) (float[4])
- `reference_frame`: Reference frame for pose (string, optional)

**Response**:
- `success`: Whether the spawn was successful (bool)
- `status_message`: Human-readable status message (string)

### /delete_entity
**Service Type**: `gazebo_msgs/srv/DeleteEntity`

**Request**:
- `name`: Name of the entity to delete (string)

**Response**:
- `success`: Whether the deletion was successful (bool)
- `status_message`: Human-readable status message (string)

### /get_entity_state
**Service Type**: `gazebo_msgs/srv/GetEntityState`

**Request**:
- `name`: Name of the entity (string)
- `reference_frame`: Reference frame for the state (string, optional)

**Response**:
- `state`: Entity state information (object)
  - `header`: Standard ROS header (std_msgs/Header)
  - `pose`: Current pose (geometry_msgs/Pose)
  - `twist`: Current velocity (geometry_msgs/Twist)
- `success`: Whether the query was successful (bool)
- `status_message`: Human-readable status message (string)

## Sensor Topics

### /lidar_scan
**Message Type**: `sensor_msgs/LaserScan`

**Message Fields**:
- `header`: Standard ROS header (std_msgs/Header)
- `angle_min`: Minimum angle of the scan [rad] (float)
- `angle_max`: Maximum angle of the scan [rad] (float)
- `angle_increment`: Angular distance between measurements [rad] (float)
- `time_increment`: Time between measurements [seconds] (float)
- `scan_time`: Time between scans [seconds] (float)
- `range_min`: Minimum range value [m] (float)
- `range_max`: Maximum range value [m] (float)
- `ranges`: Array of range measurements [m] (float[])
- `intensities`: Array of intensity measurements (float[], optional)

### /depth_camera/points
**Message Type**: `sensor_msgs/PointCloud2`

**Message Fields**:
- `header`: Standard ROS header (std_msgs/Header)
- `height`: 2D structure height (uint32)
- `width`: 2D structure width (uint32)
- `fields`: Array of available fields (array of objects)
  - `name`: Name of field (string)
  - `offset`: Offset of field in bytes (uint32)
  - `datatype`: Datatype (uint8)
  - `count`: Number of elements in field (uint32)
- `is_bigendian`: Endianness flag (bool)
- `point_step`: Size of each point in bytes (uint32)
- `row_step`: Size of row in bytes (uint32)
- `data`: Binary data blob (uint8[])
- `is_dense`: Whether points without NaNs are organized (bool)

### /depth_camera/image_raw
**Message Type**: `sensor_msgs/Image`

**Message Fields**:
- `header`: Standard ROS header (std_msgs/Header)
- `height`: Image height [pixels] (uint32)
- `width`: Image width [pixels] (uint32)
- `encoding`: Encoding type (string)
- `is_bigendian`: Endianness flag (uint8)
- `step`: Full row length in bytes (uint32)
- `data`: Actual image data (uint8[])

### /imu/data
**Message Type**: `sensor_msgs/Imu`

**Message Fields**:
- `header`: Standard ROS header (std_msgs/Header)
- `orientation`: Orientation as quaternion (geometry_msgs/Quaternion)
- `orientation_covariance`: Covariance matrix for orientation (float[9])
- `angular_velocity`: Angular velocity (geometry_msgs/Vector3)
- `angular_velocity_covariance`: Covariance matrix for angular velocity (float[9])
- `linear_acceleration`: Linear acceleration (geometry_msgs/Vector3)
- `linear_acceleration_covariance`: Covariance matrix for linear acceleration (float[9])

## Unity-ROS Bridge Messages

### /unity_joint_states
**Message Type**: `sensor_msgs/JointState`

**Message Fields**:
- `header`: Standard ROS header (std_msgs/Header)
- `name`: Array of joint names (string[])
- `position`: Array of joint positions [rad] (float[])
- `velocity`: Array of joint velocities [rad/s] (float[])
- `effort`: Array of joint efforts [Nm] (float[])

### /unity_robot_visualization
**Message Type**: `visualization_msgs/MarkerArray`

**Message Fields**:
- `markers`: Array of visualization markers (array of objects)
  - `header`: Standard ROS header (std_msgs/Header)
  - `ns`: Namespace for the marker (string)
  - `id`: Unique identifier for the marker (int32)
  - `type`: Type of marker (int32)
  - `action`: Action to take (int32)
  - `pose`: Pose of the marker (geometry_msgs/Pose)
  - `scale`: Scale of the marker (geometry_msgs/Vector3)
  - `color`: Color of the marker (std_msgs/ColorRGBA)
  - `lifetime`: Duration before marker expires (duration)
  - `frame_locked`: Whether to lock to frame (bool)
  - `points`: Array of points for line/triangle markers (array of geometry_msgs/Point)
  - `colors`: Array of colors for point visualization (array of std_msgs/ColorRGBA)
  - `text`: Text content for text markers (string)
  - `mesh_resource`: Path to mesh resource (string)
  - `mesh_use_embedded_materials`: Whether to use embedded materials (bool)