# Data Model: The Robotic Nervous System (ROS 2)

## ROS 2 Node Entity
- **Fields**: node_name (string), namespace (string), parameters (dict), lifecycle_state (enum)
- **Relationships**: Contains publishers, subscribers, services, and action servers
- **Validation rules**: node_name must follow ROS naming conventions, namespace must be valid
- **State transitions**: unconfigured → inactive → active → finalized

## ROS 2 Message Entity
- **Fields**: message_type (string), timestamp (datetime), data (dict/structured), source_node (string)
- **Relationships**: Exchanged between publishers and subscribers
- **Validation rules**: Must conform to ROS 2 message definition standards
- **State transitions**: Created → Published → Received → Processed

## ROS 2 Service Request Entity
- **Fields**: service_name (string), request_type (string), parameters (dict), client_node (string)
- **Relationships**: Sent from client to service server
- **Validation rules**: Service name must exist, parameters must match service definition
- **State transitions**: Requested → Processing → Completed/Failed

## URDF Robot Model Entity
- **Fields**: robot_name (string), links (list), joints (list), materials (dict), properties (dict)
- **Relationships**: Links connected by joints, materials applied to visuals
- **Validation rules**: Must conform to URDF schema, joints must connect valid links
- **State transitions**: Defined → Parsed → Validated → Loaded in RViz

## AI Agent Command Entity
- **Fields**: command_type (enum), target_robot (string), parameters (dict), priority (int), timestamp (datetime)
- **Relationships**: Sent to robot controllers, may receive feedback through action results
- **Validation rules**: Command must be valid for target robot, parameters must be within safe ranges
- **State transitions**: Created → Sent → Executing → Completed/Failed

## Sensor Data Entity
- **Fields**: sensor_type (string), topic_name (string), data_payload (dict), timestamp (datetime), robot_id (string)
- **Relationships**: Published by robot sensors, consumed by AI agent
- **Validation rules**: Data must conform to sensor message type, timestamp must be recent
- **State transitions**: Acquired → Processed → Published → Consumed

## ROS 2 Action Goal Entity
- **Fields**: action_name (string), goal_type (string), goal_parameters (dict), feedback_callback (function)
- **Relationships**: Sent to action server, receives feedback and result
- **Validation rules**: Action server must exist, goal parameters must be valid
- **State transitions**: Sent → Accepted → Executing → Succeeded/Aborted/Canceled