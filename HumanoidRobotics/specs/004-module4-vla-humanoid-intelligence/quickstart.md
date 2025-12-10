# Quickstart Guide: VLA Humanoid Intelligence Module

## Prerequisites

1. **System Requirements**:
   - Ubuntu 22.04 LTS
   - ROS 2 Humble Hawksbill
   - Python 3.10+
   - NVIDIA GPU with CUDA 11.8+ (for Isaac Sim)
   - 16GB+ RAM recommended

2. **Install ROS 2 Humble**:
   ```bash
   sudo apt update
   sudo apt install software-properties-common
   sudo add-apt-repository universe
   sudo apt update
   sudo apt install curl gnupg lsb-release
   curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | sudo gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   sudo apt update
   sudo apt install ros-humble-desktop
   sudo apt install python3-rosdep2
   source /opt/ros/humble/setup.bash
   ```

3. **Install Isaac Sim**:
   - Download Isaac Sim from NVIDIA Developer website
   - Follow installation instructions for your system
   - Verify installation with: `isaac-sim --version`

4. **Python Dependencies**:
   ```bash
   pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
   pip3 install transformers openai-whisper
   pip3 install opencv-python numpy scipy
   pip3 install ros2_numpy  # For ROS-Python bridge
   ```

## Setup Instructions

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd <repository-name>
   source /opt/ros/humble/setup.bash
   ```

2. **Install ROS Dependencies**:
   ```bash
   cd robotics_vla
   rosdep install --from-paths src --ignore-src -r -y
   colcon build
   source install/setup.bash
   ```

3. **Download Required Models**:
   ```bash
   # Whisper model for speech recognition
   python3 -c "import whisper; whisper.load_model('base')"

   # Download open-source LLM (example with HuggingFace)
   python3 -c "from transformers import AutoTokenizer, AutoModelForCausalLM; AutoTokenizer.from_pretrained('microsoft/DialoGPT-medium')"
   ```

## Running the VLA Pipeline

1. **Launch Isaac Sim Environment**:
   ```bash
   cd isaac_sim_workspace
   python3 -m robotics_vla.simulation.isaac_sim.simulation_env --world=humanoid_lab
   ```

2. **Start the VLA Pipeline**:
   ```bash
   ros2 launch robotics_vla vla_pipeline.launch.py
   ```

3. **Run a Simple Voice Command**:
   ```bash
   ros2 run robotics_vla examples/simple_voice_command.py --command="Move forward 2 meters"
   ```

## Chapter Examples

### Chapter 1: VLA Fundamentals
```bash
# Run the VLA architecture demonstration
ros2 run robotics_vla examples/vla_fundamentals_demo.py
```

### Chapter 2: Voice-to-Action
```bash
# Test voice command processing
ros2 run robotics_vla examples/voice_to_action_demo.py
```

### Chapter 3: Cognitive Planning
```bash
# Generate and execute a multi-step plan
ros2 run robotics_vla examples/cognitive_planning_demo.py --task="Go to kitchen and pick up red cup"
```

### Chapter 4: Capstone Project
```bash
# Run the complete capstone demonstration
ros2 run robotics_vla examples/capstone_demo.py
```

## Troubleshooting

**Issue**: Isaac Sim not launching
- **Solution**: Verify GPU drivers and CUDA installation: `nvidia-smi`

**Issue**: ROS 2 nodes not communicating
- **Solution**: Check ROS_DOMAIN_ID and ensure all terminals source the same setup.bash

**Issue**: Whisper model not loading
- **Solution**: Check internet connection and download models manually if needed

**Issue**: Object detection failing
- **Solution**: Verify camera calibration and lighting conditions in simulation

## Assessment

To run the built-in assessments:
```bash
# Knowledge check for Chapter 1
python3 -m robotics_vla.educational.assessment.knowledge_checks --chapter=1

# Project evaluation for capstone
python3 -m robotics_vla.educational.assessment.project_evaluations --project=capstone
```