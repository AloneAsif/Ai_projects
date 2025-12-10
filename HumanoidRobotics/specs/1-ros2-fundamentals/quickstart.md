# Quickstart Guide: The Robotic Nervous System (ROS 2)

## Prerequisites

- Ubuntu 22.04 (for ROS 2 Humble) or Ubuntu 23.04 (for ROS 2 Iron)
- ROS 2 Humble Hawksbill or Iron Irwini installed
- Python 3.8+
- Basic understanding of Python programming

## Installation

1. **Install ROS 2** (if not already installed):
   ```bash
   # For ROS 2 Humble on Ubuntu 22.04
   sudo apt update
   sudo apt install software-properties-common
   sudo add-apt-repository universe
   sudo apt update && sudo apt install curl gnupg lsb-release
   curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | sudo apt-key add -
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/lib/sudo apt-key.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   sudo apt update
   sudo apt install ros-humble-desktop
   sudo apt install python3-rosdep2
   source /opt/ros/humble/setup.bash
   ```

2. **Set up ROS 2 environment**:
   ```bash
   echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

3. **Create workspace**:
   ```bash
   mkdir -p ~/ros2_ws/src
   cd ~/ros2_ws
   colcon build
   source install/setup.bash
   ```

## Running the Examples

### Chapter 1: ROS 2 Overview
No code to run - conceptual chapter with diagrams.

### Chapter 2: Hands-On ROS 2 (rclpy)

1. **Publisher/Subscriber Example**:
   ```bash
   # Navigate to the workspace
   cd ~/ros2_ws/src
   # Create the package (if not already created)
   ros2 pkg create --build-type ament_python publisher_subscriber
   # Copy the example files to the package
   cd ~/ros2_ws
   colcon build --packages-select publisher_subscriber
   source install/setup.bash
   # Run the publisher
   ros2 run publisher_subscriber talker
   # In another terminal, run the subscriber
   ros2 run publisher_subscriber listener
   ```

2. **Service Example**:
   ```bash
   # Run the service server
   ros2 run services add_two_ints_server
   # In another terminal, run the client
   ros2 run services add_two_ints_client 1 2
   ```

### Chapter 3: Python AI Agent → ROS Bridge

1. **Run the AI agent**:
   ```bash
   cd ~/ros2_ws
   source install/setup.bash
   ros2 run ai_agent ai_ros_bridge
   ```

2. **Send commands to the robot**:
   ```bash
   ros2 run ai_agent robot_commander
   ```

### Chapter 4: URDF for Humanoids

1. **View the URDF model in RViz**:
   ```bash
   # Launch RViz with the humanoid model
   ros2 launch urdf_display.launch.py model:='$(find package_name)/urdf/humanoid_model.urdf'
   ```

## Building the Project

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

## Testing the Examples

```bash
cd ~/ros2_ws
colcon build --packages-select publisher_subscriber services ai_agent
source install/setup.bash
colcon test --packages-select publisher_subscriber services ai_agent
```

## Troubleshooting

1. **Package not found**: Make sure to source the setup.bash file after building
2. **Permission errors**: Check that ROS 2 is properly installed and environment is sourced
3. **Python import errors**: Ensure Python packages are installed in the correct environment
4. **RViz not displaying model**: Verify URDF file path and syntax

## Next Steps

After completing these examples, you should have a solid understanding of:
- ROS 2 node communication patterns (publishers, subscribers, services)
- How to integrate AI agents with ROS 2 systems
- How to create and visualize robot models using URDF
- Best practices for ROS 2 development in Python