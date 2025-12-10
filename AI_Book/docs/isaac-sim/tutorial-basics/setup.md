---
sidebar_label: 'Isaac Sim Setup Guide'
sidebar_position: 1
---

# Isaac Sim Setup Guide

## Prerequisites

Before starting with Isaac Sim VSLAM and Navigation, ensure you have the following prerequisites installed:

### Hardware Requirements
- NVIDIA GPU with RTX capabilities (RTX 2070 or higher recommended)
- 16GB+ RAM for handling large simulation scenes
- 100GB+ free disk space for Isaac Sim installation
- Ubuntu 22.04 LTS operating system

### Software Requirements
- ROS 2 Humble Hawksbill
- Isaac Sim (with educational licensing)
- Isaac ROS packages: isaac_ros_visual_slam, isaac_ros_image_pipeline, isaac_ros_common
- Nav2 packages: ros-humble-navigation2, ros-humble-nav2-bringup
- CUDA toolkit compatible with your GPU
- Docker (optional, for containerized environments)

## Isaac Sim Installation

### Step 1: Download Isaac Sim
1. Visit the NVIDIA Developer website
2. Download Isaac Sim for your platform
3. Ensure you have educational licensing if required

### Step 2: Install Isaac Sim
```bash
# Extract the downloaded archive
tar -xzf isaac-sim-YYYY.X.X-linux.tar.gz

# Navigate to the extracted directory
cd isaac-sim-YYYY.X.X

# Run the setup script
./setup.sh
```

### Step 3: Verify Installation
```bash
# Launch Isaac Sim
./isaac-sim.sh
```

## Isaac ROS Installation

### Install Isaac ROS Packages
```bash
# Add NVIDIA package repository
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.nvidia.com/ubuntu/$(lsb_release -cs)/arm64 /"
sudo apt update

# Install Isaac ROS packages
sudo apt install -y ros-humble-isaac-ros-common
sudo apt install -y ros-humble-isaac-ros-visual-slam
sudo apt install -y ros-humble-isaac-ros-image-pipeline
```

## ROS 2 Workspace Setup

### Create Workspace for Isaac Sim Projects
```bash
# Create workspace directory
mkdir -p ~/isaac_ws/src
cd ~/isaac_ws

# Source ROS 2 Humble
source /opt/ros/humble/setup.bash

# Build the workspace
colcon build --symlink-install
```

### Environment Setup
Add the following to your `~/.bashrc`:
```bash
# ROS 2 Humble setup
source /opt/ros/humble/setup.bash

# Isaac workspace setup
source ~/isaac_ws/install/setup.bash

# Isaac Sim setup (adjust path as needed)
export ISAAC_SIM_PATH=/path/to/isaac-sim
export PATH=$ISAAC_SIM_PATH:$PATH
```

## Docker Setup (Optional)

### Isaac Sim Docker Environment

Create the Isaac Sim environment Dockerfile at `docker/isaac_sim_env/Dockerfile`:

```dockerfile
FROM nvidia/cudagl:12.0-devel-ubuntu22.04

# Install Isaac Sim dependencies
RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-dev \
    build-essential \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install Isaac ROS packages
RUN apt-get update && apt-get install -y \
    ros-humble-isaac-ros-common \
    ros-humble-isaac-ros-visual-slam \
    ros-humble-isaac-ros-image-pipeline \
    && rm -rf /var/lib/apt/lists/*

# Set up workspace
WORKDIR /workspace

# Install Isaac Sim Python dependencies
RUN pip3 install --upgrade pip && \
    pip3 install numpy scipy matplotlib
```

### ROS 2 Docker Environment

Create the ROS 2 environment Dockerfile at `docker/ros2_env/Dockerfile`:

```dockerfile
FROM osrf/ros:humble-desktop

# Install Isaac ROS packages
RUN apt-get update && apt-get install -y \
    ros-humble-isaac-ros-common \
    ros-humble-isaac-ros-visual-slam \
    ros-humble-isaac-ros-image-pipeline \
    ros-humble-navigation2 \
    ros-humble-nav2-bringup \
    && rm -rf /var/lib/apt/lists/*

# Set up workspace
WORKDIR /workspace

# Install additional dependencies
RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-dev \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

RUN pip3 install numpy scipy matplotlib
```

### Building and Running Docker Containers

```bash
# Build Isaac Sim container
cd docker/isaac_sim_env
docker build -t isaac-sim-env .

# Run Isaac Sim container with GPU support
docker run --gpus all -it --rm \
  --env="DISPLAY" \
  --env="QT_X11_NO_MITSHM=1" \
  --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  --privileged \
  --name isaac-sim-container \
  isaac-sim-env

# Build ROS 2 container
cd docker/ros2_env
docker build -t ros2-env .

# Run ROS 2 container
docker run -it --rm --name ros2-container ros2-env
```

## Troubleshooting Common Issues

### GPU Not Detected
- Ensure NVIDIA drivers are properly installed
- Verify CUDA is working with `nvidia-smi`
- Check Isaac Sim requirements match your GPU

### Performance Issues
- Reduce scene complexity during development
- Adjust rendering quality settings
- Ensure sufficient RAM is available

### ROS 2 Integration Problems
- Verify ROS 2 Humble is properly sourced
- Check that Isaac ROS packages are installed
- Confirm network configuration allows ROS communication

## Next Steps

After completing the setup, proceed to:
1. [Chapter 1: Isaac Sim Perception Fundamentals](../ch1-perception-fundamentals)
2. [Chapter 2: Isaac ROS VSLAM Pipeline](../ch2-vslam-pipeline)
3. [Chapter 3: Nav2 Navigation for Humanoids](../ch3-nav2-navigation)