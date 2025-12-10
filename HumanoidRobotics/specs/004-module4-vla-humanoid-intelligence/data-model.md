# Data Model: VLA Humanoid Intelligence Module

## Core Entities

### VLAPipeline
- **Attributes**:
  - pipeline_id (string, unique)
  - status (enum: initialized, processing, completed, failed)
  - created_at (timestamp)
  - last_updated (timestamp)
- **Relationships**: Contains VisionProcessor, LanguageProcessor, ActionExecutor
- **Validation**: Must have valid components before execution

### VisionProcessor
- **Attributes**:
  - processor_id (string, unique)
  - camera_feed (stream reference)
  - detected_objects (list of ObjectDetection)
  - confidence_threshold (float, 0.0-1.0)
- **State Transitions**: idle → processing → results_ready → idle

### LanguageProcessor
- **Attributes**:
  - processor_id (string, unique)
  - speech_input (audio stream)
  - text_output (string)
  - intent (string)
  - task_graph (JSON structure)
- **State Transitions**: idle → listening → processing → task_generated → idle

### ActionExecutor
- **Attributes**:
  - executor_id (string, unique)
  - current_task (string)
  - task_status (enum: pending, executing, completed, failed)
  - robot_state (JSON structure)
- **State Transitions**: idle → planning → executing → completed/failed → idle

### ObjectDetection
- **Attributes**:
  - object_id (string, unique)
  - class_name (string)
  - confidence (float, 0.0-1.0)
  - position_3d (x, y, z coordinates)
  - bounding_box (2D coordinates)
- **Validation**: Confidence must be > threshold to be considered valid

### TaskGraph
- **Attributes**:
  - graph_id (string, unique)
  - nodes (list of TaskNode)
  - edges (list of dependencies)
  - start_node (string)
  - end_node (string)
- **Validation**: Must form a valid directed acyclic graph

### TaskNode
- **Attributes**:
  - node_id (string, unique)
  - task_type (enum: navigation, manipulation, perception, wait)
  - parameters (JSON structure)
  - prerequisites (list of node_ids)
- **Validation**: Parameters must match task_type requirements

### SimulationEnvironment
- **Attributes**:
  - env_id (string, unique)
  - world_config (file reference)
  - robot_model (URDF/SDF reference)
  - sensors_config (JSON structure)
  - physics_properties (JSON structure)
- **Validation**: Robot model must be compatible with environment

### StudentProgress
- **Attributes**:
  - student_id (string, unique)
  - chapter_completed (integer)
  - tasks_completed (list of task_ids)
  - assessment_scores (list of scores)
  - completion_time (timestamp)
- **Validation**: Scores must be between 0 and 100

## Relationships
- VLAPipeline "has one" VisionProcessor
- VLAPipeline "has one" LanguageProcessor
- VLAPipeline "has one" ActionExecutor
- VisionProcessor "produces many" ObjectDetection
- LanguageProcessor "generates one" TaskGraph
- TaskGraph "contains many" TaskNode
- ActionExecutor "executes many" TaskNode
- SimulationEnvironment "hosts one" VLAPipeline
- StudentProgress "tracks many" VLAPipeline executions