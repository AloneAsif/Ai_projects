# Quickstart Guide: Simulation Fundamentals with Gazebo and Unity

## Prerequisites

### System Requirements
- Ubuntu 22.04 LTS or Windows 10/11 with WSL2
- 8GB+ RAM, 4+ CPU cores recommended
- NVIDIA GPU with Unity support (optional but recommended)

### Software Dependencies
1. **ROS 2 Humble Hawksbill** (or later LTS version)
   ```bash
   # Ubuntu installation
   sudo apt update && sudo apt install curl gnupg lsb-release
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   sudo apt update
   sudo apt install ros-humble-desktop
   ```

2. **Gazebo Garden** (or Ignition Fortress)
   ```bash
   sudo apt install ignition-garden
   # Or for newer Gazebo Garden:
   sudo apt install gz-sim7
   ```

3. **Unity LTS 2022.3.x** or later
   - Download from Unity Hub
   - Install with Linux Build Support if targeting Linux

4. **Unity Robotics Open Source (UROS) Package**
   - Available via Unity Package Manager

## Setup Process

### 1. Clone the Repository
```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Install ROS 2 Packages
```bash
# Source ROS 2
source /opt/ros/humble/setup.bash
# Or add to your .bashrc:
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc

# Install additional ROS 2 packages
sudo apt install ros-humble-gazebo-ros-pkgs ros-humble-robot-state-publisher ros-humble-joint-state-publisher
```

### 3. Configure Unity Project
1. Open Unity Hub and add the project from `unity/` directory
2. Import the UROS package via Package Manager
3. Install Unity URDF Importer from Package Manager
4. Verify all dependencies are resolved

### 4. Environment Setup
```bash
# Create a workspace
mkdir -p ~/simulation_ws/src
cd ~/simulation_ws
colcon build
source install/setup.bash
```

## Running the Examples

### Chapter 1: Digital Twin Fundamentals
1. Review the theoretical content in `docs/chapter1-theory.md`
2. No simulation required for this chapter

### Chapter 2: Gazebo Physics Simulation
```bash
# Launch the basic physics environment
cd ~/simulation_ws
source install/setup.bash
ros2 launch simulation_examples basic_physics.launch.py
```

### Chapter 3: Sensor Simulation
```bash
# Launch the sensor simulation environment
cd ~/simulation_ws
source install/setup.bash
ros2 launch simulation_examples sensor_simulation.launch.py
```

### Chapter 4: Unity Digital Twin
1. In Unity Editor, open the scene from `unity/Scenes/DigitalTwin.unity`
2. Configure the ROS connection settings (IP, port)
3. Run the scene to see synchronized visualization

## Troubleshooting

### Common Issues

**Issue**: Gazebo fails to start with graphics errors
**Solution**:
```bash
# Run with software rendering
export LIBGL_ALWAYS_SOFTWARE=1
gz sim
# Or ensure proper GPU drivers are installed
```

**Issue**: Unity-ROS bridge connection fails
**Solution**: Check IP addresses and firewall settings
- Verify ROS 2 master is accessible
- Check network configuration between Unity and ROS systems

**Issue**: Humanoid robot falls through ground
**Solution**:
- Verify URDF collision meshes are properly defined
- Check physics parameters (gravity, friction coefficients)
- Ensure proper joint limits and damping

## Next Steps

1. Complete Chapter 1 theory
2. Run basic physics simulation (Chapter 2)
3. Add sensors and observe data (Chapter 3)
4. Connect to Unity visualization (Chapter 4)
5. Experiment with custom environments and robots