# Quickstart Guide: Isaac Sim VSLAM and Navigation Module

## Overview
This guide will help you set up and run the Isaac Sim VSLAM and Navigation educational module. Follow these steps to get started with the complete pipeline from simulation to navigation.

## Prerequisites

### Hardware Requirements
- NVIDIA GPU with RTX capabilities (RTX 2070 or higher recommended)
- 16GB+ RAM (32GB recommended for complex scenes)
- 100GB+ free disk space for Isaac Sim and datasets

### Software Requirements
- Ubuntu 22.04 LTS
- ROS 2 Humble Hawksbill or Iron Irwini
- Isaac Sim (with proper educational licensing)
- Isaac ROS packages
- NVIDIA GPU drivers (520+)
- Docker (optional, for containerized environments)

## Installation Steps

### 1. System Setup
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install NVIDIA drivers (if not already installed)
sudo apt install nvidia-driver-535

# Reboot after driver installation
sudo reboot
```

### 2. ROS 2 Installation
```bash
# Add ROS 2 repository
sudo apt update && sudo apt install -y curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt install ros-humble-desktop
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential

# Initialize rosdep
sudo rosdep init
rosdep update

# Source ROS 2 environment
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 3. Isaac Sim Installation
```bash
# Download and install Isaac Sim from NVIDIA Developer website
# Follow the official installation guide for your system configuration
# Ensure proper licensing is configured for educational use

# Verify Isaac Sim installation
cd /path/to/isaac-sim
./runheadless.py --version
```

### 4. Isaac ROS Setup
```bash
# Create ROS 2 workspace
mkdir -p ~/isaac_ws/src
cd ~/isaac_ws

# Install Isaac ROS dependencies
sudo apt update
sudo apt install -y python3-pip
pip3 install wheel

# Clone Isaac ROS packages
cd ~/isaac_ws/src
git clone -b ros2 https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_visual_slam.git
git clone -b ros2 https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_image_pipeline.git
git clone -b ros2 https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common.git

# Install additional dependencies
rosdep install --from-paths src --ignore-src -r -y

# Build the workspace
cd ~/isaac_ws
colcon build --symlink-install
source install/setup.bash
```

### 5. Nav2 Setup
```bash
# Install Nav2 packages
sudo apt install ros-humble-navigation2 ros-humble-nav2-bringup

# Verify installation
ros2 launch nav2_bringup navigation_launch.py --show-args
```

## Running the Examples

### Chapter 1: Isaac Sim Perception Fundamentals
```bash
# Navigate to the chapter directory
cd ~/isaac_ws/src/isaac_sim_vslam_nav/chapter_1_perception

# Launch Isaac Sim with a basic scene
./scripts/setup_isaac_sim.sh

# Generate a synthetic dataset
cd synthetic_data_gen
python3 generate_dataset.py --scene basic_office --output_dir datasets/office_dataset --count 100
```

### Chapter 2: Isaac ROS VSLAM Pipeline
```bash
# Source the workspace
cd ~/isaac_ws
source install/setup.bash

# Launch the VSLAM pipeline
cd ~/isaac_ws/src/isaac_sim_vslam_nav/chapter_2_vslam
ros2 launch isaac_ros_visual_slam visual_slam.launch.py

# In another terminal, play back sensor data or run simulation
# Monitor the pose estimates being published to ROS topics
ros2 topic echo /visual_slam/tracking/pose
```

### Chapter 3: Nav2 Navigation for Humanoids
```bash
# Source the workspace
cd ~/isaac_ws
source install/setup.bash

# Launch Nav2 with humanoid-specific configurations
cd ~/isaac_ws/src/isaac_sim_vslam_nav/chapter_3_navigation
ros2 launch nav2_bringup navigation_launch.py \
  params_file:=config/humanoid_nav2_params.yaml \
  map:=maps/humanoid_environment.yaml

# Send a navigation goal
ros2 run nav2_msgs navigate_to_pose \
  --ros-args -p pose='{header: {frame_id: "map"}, pose: {position: {x: 1.0, y: 1.0, z: 0.0}, orientation: {w: 1.0}}}'
```

## Validation Steps

### Verify Installation
```bash
# Check Isaac Sim functionality
cd /path/to/isaac-sim
./runheadless.py --test

# Check ROS 2 functionality
source ~/isaac_ws/install/setup.bash
ros2 topic list

# Check Isaac ROS packages
ros2 pkg list | grep isaac
```

### Run Validation Tests
```bash
# Navigate to validation scripts
cd ~/isaac_ws/src/isaac_sim_vslam_nav

# Run the complete validation suite
./scripts/validation_tests.sh

# Expected outputs:
# - Isaac Sim renders at 30+ FPS
# - VSLAM produces pose estimates with <5cm accuracy
# - Nav2 computes paths within 2 seconds
# - All examples reproduce successfully
```

## Troubleshooting Common Issues

### Isaac Sim Not Launching
- Ensure NVIDIA GPU drivers are properly installed
- Check that Isaac Sim has proper licensing
- Verify RTX capabilities of your GPU

### VSLAM Pipeline Not Tracking
- Verify camera calibration parameters
- Check that sensor data is being published to correct topics
- Ensure sufficient visual features in the environment

### Nav2 Path Planning Failing
- Verify map is properly loaded
- Check that costmaps are being updated
- Ensure humanoid-specific constraints are properly configured

### Performance Issues
- Reduce simulation complexity for lower-end hardware
- Adjust rendering settings in Isaac Sim
- Use lighter-weight VSLAM configurations

## Next Steps

1. Complete the tutorials in order (Chapter 1 → Chapter 2 → Chapter 3)
2. Experiment with different scene configurations
3. Modify pipeline parameters to understand their effects
4. Create custom navigation scenarios
5. Validate your results against the success criteria

## Support Resources

- Official Isaac Sim Documentation
- ROS 2 Humble Tutorials
- Nav2 Documentation
- Isaac ROS GitHub Repositories
- Troubleshooting guide in documentation/