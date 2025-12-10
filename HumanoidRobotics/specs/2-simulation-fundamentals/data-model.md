# Data Model: Simulation Fundamentals with Gazebo and Unity

## Simulation Environment

**Description**: Represents the virtual world containing physics properties, lighting, textures, and obstacles

**Attributes**:
- `id`: Unique identifier for the environment (string)
- `name`: Display name of the environment (string)
- `physics_properties`: Object containing gravity, friction coefficients, etc.
  - `gravity`: 3D vector [x, y, z] representing gravitational acceleration (float[3])
  - `linear_damping`: Coefficient for linear motion damping (float)
  - `angular_damping`: Coefficient for angular motion damping (float)
- `lighting_setup`: Object containing lighting configuration
  - `ambient_light`: Ambient light intensity (float)
  - `directional_light`: Directional light properties (object)
    - `direction`: Light direction vector (float[3])
    - `intensity`: Light intensity (float)
- `obstacles`: Array of obstacle objects in the environment (array of objects)
  - `type`: Type of obstacle (static/dynamic) (string)
  - `position`: Position in 3D space (float[3])
  - `shape`: Geometric shape (box, sphere, cylinder, mesh) (string)
  - `dimensions`: Size parameters based on shape (float[3])

**Relationships**:
- Contains multiple Robot Models
- Contains multiple Sensor Configurations

**Validation Rules**:
- Must have valid URDF/SDF and Unity scene references
- Gravity vector magnitude must be within realistic range (0.1 to 20.0 m/s²)
- Lighting values must be non-negative

## Humanoid Robot Model

**Description**: Represents the digital twin with URDF/FBX format, joints, links, and physical properties

**Attributes**:
- `id`: Unique identifier for the robot model (string)
- `name`: Display name of the robot (string)
- `urdf_path`: File path to URDF description (string)
- `joint_configuration`: Array of joint objects (array of objects)
  - `name`: Joint name (string)
  - `type`: Joint type (revolute, prismatic, fixed, etc.) (string)
  - `limits`: Joint limits (object)
    - `lower`: Lower limit (float)
    - `upper`: Upper limit (float)
    - `effort`: Maximum effort (float)
    - `velocity`: Maximum velocity (float)
- `physical_properties`: Object containing mass and inertia data (object)
  - `mass`: Total mass of the link (float)
  - `inertia`: Inertia tensor (float[6] - Ixx, Iyy, Izz, Ixy, Ixz, Iyz)
  - `collision_shapes`: Array of collision geometry definitions (array of objects)
- `visualization`: Associated Unity prefab and material information (object)
  - `prefab_path`: Path to Unity prefab (string)
  - `materials`: Array of material references (array of strings)

**Relationships**:
- Belongs to one Simulation Environment
- Contains multiple Sensor Attachments

**Validation Rules**:
- URDF must load without physics errors in Gazebo
- Joint limits must be physically realistic
- Mass values must be positive

## Sensor Configuration

**Description**: Represents a sensor attached to a robot or environment with specific parameters

**Attributes**:
- `id`: Unique identifier for the sensor (string)
- `name`: Display name of the sensor (string)
- `type`: Type of sensor (lidar, depth_camera, imu, etc.) (string)
- `parent_link`: Link to which the sensor is attached (string)
- `position`: Position relative to parent link (float[3])
- `orientation`: Orientation relative to parent link (float[4] - quaternion)
- `topic_config`: ROS 2 topic configuration (object)
  - `topic_name`: Name of the ROS 2 topic (string)
  - `message_type`: ROS 2 message type (string)
  - `update_rate`: Rate at which sensor publishes data (float)
- `sensor_properties`: Type-specific properties (object)
  - `range`: For range sensors (float)
  - `fov`: Field of view (float)
  - `resolution`: Spatial resolution (float)

**Relationships**:
- Attached to one Robot Model
- Publishes to one or more ROS 2 Topics

**Validation Rules**:
- Must conform to ROS 2 message standards
- Update rate must be within realistic range
- Position and orientation must be valid transforms

## Physics Parameters

**Description**: Represents physical properties that affect simulation behavior

**Attributes**:
- `id`: Unique identifier for the parameter set (string)
- `name`: Display name of the parameter set (string)
- `gravity`: 3D vector for gravitational acceleration (float[3])
- `friction_coefficients`: Object containing friction data (object)
  - `static`: Static friction coefficient (float)
  - `dynamic`: Dynamic friction coefficient (float)
- `damping_coefficients`: Object containing damping data (object)
  - `linear`: Linear damping coefficient (float)
  - `angular`: Angular damping coefficient (float)
- `simulation_rate`: Physics update rate (Hz) (float)

**Relationships**:
- Applied to one Simulation Environment

**Validation Rules**:
- Must use SI units consistently
- Friction coefficients must be non-negative
- Simulation rate must be positive and reasonable (typically 100-1000 Hz)

## ROS 2 Topic

**Description**: Represents a communication channel in the ROS 2 system

**Attributes**:
- `id`: Unique identifier for the topic (string)
- `name`: Full topic name with namespace (string)
- `message_type`: ROS 2 message type (string)
- `qos_profile`: Quality of service settings (object)
  - `reliability`: Reliability policy (string)
  - `durability`: Durability policy (string)
  - `history`: History policy (string)
- `publishers`: List of nodes publishing to this topic (array of strings)
- `subscribers`: List of nodes subscribing to this topic (array of strings)

**Relationships**:
- Connected to multiple Sensor Configurations
- Connected to multiple Visualization Components

**Validation Rules**:
- Topic names must follow ROS 2 naming conventions
- Message types must be valid ROS 2 message types
- QoS settings must be compatible between publishers and subscribers