---
sidebar_label: 'Quick Reference'
sidebar_position: 2
---

# Quick Reference Guide

## Common Commands

### Isaac Sim Commands
```bash
# Launch Isaac Sim
./isaac-sim.sh

# Run headless simulation
./isaac-sim.sh --/headless=true

# Load specific scene
./isaac-sim.sh --summary=0 --/Isaac/IsaacApp/window/w=1920 --/Isaac/IsaacApp/window/h=1080
```

### ROS 2 Commands for Isaac Sim
```bash
# Source workspace
source ~/isaac_ws/install/setup.bash

# Launch VSLAM pipeline
ros2 launch isaac_ros_visual_slam visual_slam.launch.py

# Launch Nav2 navigation
ros2 launch nav2_bringup navigation_launch.py

# View topics
ros2 topic list
ros2 topic echo /visual_slam/pose

# Check services
ros2 service list
```

### Simulation Commands
```bash
# Start simulation
omni.isaac.core.World().play()

# Stop simulation
omni.isaac.core.World().stop()

# Reset simulation
omni.isaac.core.World().reset()
```

## Key Parameters

### VSLAM Parameters
- `track_features_max_corners`: 1000 (max features to track)
- `optimization_num_iterations`: 5 (optimization iterations)
- `map_size`: 1000 (max landmarks in map)
- `loop_closure_min_inliers`: 20 (min matches for loop closure)

### Navigation Parameters
- `max_step_size`: 0.3m (maximum humanoid step size)
- `robot_radius`: 0.25m (collision radius)
- `xy_goal_tolerance`: 0.1m (position tolerance)
- `yaw_goal_tolerance`: 0.1rad (orientation tolerance)

## Common API Patterns

### Isaac Sim Python API
```python
import omni
from omni.isaac.core import World
from omni.isaac.core.utils.stage import add_reference_to_stage

# Initialize world
world = World(stage_units_in_meters=1.0)

# Add objects to simulation
add_reference_to_stage(
    usd_path="/path/to/asset.usd",
    prim_path="/World/MyObject"
)

# Run simulation
world.reset()
for i in range(100):
    world.step(render=True)
```

### ROS 2 Integration
```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from geometry_msgs.msg import PoseStamped

class IsaacSimBridge(Node):
    def __init__(self):
        super().__init__('isaac_sim_bridge')

        # Publishers
        self.image_pub = self.create_publisher(Image, '/camera/image_raw', 10)
        self.pose_pub = self.create_publisher(PoseStamped, '/vslam/pose', 10)

        # Timers
        self.timer = self.create_timer(0.033, self.publish_data)  # 30 Hz
```

## Performance Tips

### Simulation Optimization
- Use Level of Detail (LOD) for complex objects
- Reduce physics substeps for faster simulation
- Use occlusion culling for large scenes
- Optimize material complexity

### VSLAM Optimization
- Adjust feature detection based on scene complexity
- Use appropriate image resolution (balance quality vs. performance)
- Optimize tracking parameters for specific environments
- Enable GPU acceleration where possible

### Navigation Optimization
- Use appropriate costmap resolution (balance precision vs. performance)
- Optimize controller frequency based on robot capabilities
- Adjust inflation parameters for safety vs. efficiency
- Use appropriate path planners for environment type

## Troubleshooting

### Simulation Issues
- **Slow performance**: Reduce scene complexity or increase hardware resources
- **Crashes**: Check GPU memory usage and reduce scene complexity
- **Artifacts**: Verify lighting and material configurations

### VSLAM Issues
- **Tracking loss**: Improve lighting or add texture to environment
- **Drift**: Enable loop closure or improve calibration
- **Poor accuracy**: Verify calibration and adjust parameters

### Navigation Issues
- **Path planning failures**: Check costmap configuration and inflation
- **Oscillation**: Adjust controller parameters
- **Safety violations**: Increase inflation radius and safety margins

## Common Error Messages

- `CUDA error`: Verify GPU and CUDA installation
- `No transform found`: Check TF tree and frame names
- `Failed to find path`: Verify costmap and goal position
- `Tracking lost`: Check visual features and lighting conditions