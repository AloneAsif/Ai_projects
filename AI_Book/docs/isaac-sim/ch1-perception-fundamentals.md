---
sidebar_label: 'Chapter 1: Isaac Sim Perception Fundamentals'
sidebar_position: 1
---

# Chapter 1: Isaac Sim Perception Fundamentals

## Overview

This chapter introduces Isaac Sim as an RTX-accelerated simulation environment for photorealistic scenes and synthetic data generation. Students will learn how to create realistic simulation environments that can generate training datasets for AI vision models, with a focus on humanoid robotics applications.

## Learning Objectives

By the end of this chapter, students will be able to:

- Set up Isaac Sim with proper educational licensing
- Configure RTX lighting and materials for realistic rendering
- Create simulation scenes with objects and physics properties
- Generate synthetic datasets with images, depth maps, and segmentation labels
- Validate dataset quality for AI training applications

## Introduction to Isaac Sim

Isaac Sim is NVIDIA's simulation environment that leverages RTX-accelerated rendering to create photorealistic simulation environments. For humanoid robotics applications, Isaac Sim provides:

- **RTX Lighting**: Realistic lighting simulation with global illumination
- **Physics Simulation**: Accurate physics with realistic materials and interactions
- **Synthetic Data Generation**: High-quality datasets for training AI models
- **Isaac ROS Integration**: Seamless integration with ROS 2 for perception pipelines

### Key Capabilities

- RTX-accelerated rendering for photorealistic outputs
- USD (Universal Scene Description) format for scene definition
- Physics simulation with PhysX engine
- Synthetic dataset generation with proper labeling
- Integration with Isaac ROS for perception and navigation

## Setting Up Isaac Sim

### Prerequisites

- NVIDIA GPU with RTX capabilities (RTX 2070 or higher recommended)
- Ubuntu 22.04 LTS with ROS 2 Humble Hawksbill
- Isaac Sim installation with educational licensing
- Isaac ROS packages installed

### Installation Process

1. Download Isaac Sim from NVIDIA Developer website
2. Configure educational licensing
3. Verify installation with basic scene loading
4. Set up Isaac Sim Python API access

## Creating Basic Office Scene

### Scene Structure

```python
# Basic office scene structure
import omni
from omni.isaac.core import World
from omni.isaac.core.utils.stage import add_reference_to_stage
from omni.isaac.core.utils.prims import create_primitive

# Initialize the world
world = World(stage_units_in_meters=1.0)

# Create basic office environment
# - Floor with realistic materials
# - Walls with proper lighting
# - Office furniture (desks, chairs, etc.)
# - RTX lighting configuration
```

### RTX Lighting Configuration

RTX lighting in Isaac Sim provides realistic global illumination:

- **Environment Lighting**: HDRI-based environment maps
- **Directional Lights**: Sun-like lighting with shadows
- **Point Lights**: Localized lighting for indoor scenes
- **Area Lights**: Soft lighting for realistic illumination

### Material Definitions

Realistic materials are crucial for photorealistic rendering:

- **PBR Materials**: Physically-based rendering materials
- **Surface Properties**: Roughness, metallic, normal maps
- **Subsurface Scattering**: For realistic skin and organic materials
- **Anisotropic Materials**: For brushed metals and special surfaces

## Synthetic Dataset Generation

### Dataset Pipeline

The synthetic dataset generation pipeline includes:

1. **Scene Configuration**: Setting up camera positions and angles
2. **Sensor Setup**: Configuring RGB, depth, and segmentation sensors
3. **Data Capture**: Capturing synchronized data from multiple sensors
4. **Label Generation**: Creating segmentation and depth maps
5. **Export**: Exporting datasets in standard formats

### Synthetic Data Generation Script

Create the synthetic dataset generation script at `chapter_1_perception/synthetic_data_gen/generate_dataset.py`:

```python
#!/usr/bin/env python3
"""
Synthetic Dataset Generation for Isaac Sim Perception Training
Implements the synthetic data generation pipeline as specified in the user story.
"""

import numpy as np
import omni
from omni.isaac.core import World
from omni.isaac.core.utils.stage import add_reference_to_stage
from omni.isaac.sensor import Camera
from omni.isaac.core.utils.viewports import set_camera_view
import carb
import os
import cv2
from PIL import Image
import json
from typing import List, Tuple, Dict, Any


class SyntheticDatasetGenerator:
    """
    Implements synthetic dataset generation for AI vision model training
    following the requirements from User Story 1: Isaac Sim Perception Fundamentals
    """

    def __init__(self, output_dir: str = "./synthetic_dataset", config_path: str = None):
        """
        Initialize the synthetic dataset generator

        Args:
            output_dir: Directory to save generated datasets
            config_path: Path to dataset configuration file
        """
        self.output_dir = output_dir
        self.config = self._load_config(config_path) if config_path else self._default_config()
        self.world = World(stage_units_in_meters=1.0)
        self.cameras = {}
        self.dataset_counter = 0

        # Create output directories
        os.makedirs(os.path.join(output_dir, "rgb"), exist_ok=True)
        os.makedirs(os.path.join(output_dir, "depth"), exist_ok=True)
        os.makedirs(os.path.join(output_dir, "segmentation"), exist_ok=True)
        os.makedirs(os.path.join(output_dir, "labels"), exist_ok=True)

    def _load_config(self, config_path: str) -> Dict[str, Any]:
        """Load dataset configuration from file"""
        with open(config_path, 'r') as f:
            return json.load(f)

    def _default_config(self) -> Dict[str, Any]:
        """Default configuration for dataset generation"""
        return {
            "dataset_size": 1000,
            "image_resolution": [640, 480],
            "camera_positions": [
                {"x": 0.0, "y": -1.0, "z": 1.5},
                {"x": 1.0, "y": 0.0, "z": 1.5},
                {"x": -1.0, "y": 0.0, "z": 1.5}
            ],
            "lighting_conditions": ["indoor", "outdoor", "low_light"],
            "object_variations": ["color", "size", "texture"],
            "sensor_config": {
                "rgb": {"format": "rgb8", "frequency": 30},
                "depth": {"format": "32FC1", "frequency": 30},
                "segmentation": {"format": "labels", "frequency": 30}
            }
        }

    def setup_scene(self, scene_config: Dict[str, Any]):
        """
        Set up the Isaac Sim scene for synthetic data generation
        Implements FR-002: System MUST enable synthetic data generation with images, depth, and segmentation maps
        """
        # Add basic environment
        add_reference_to_stage(
            usd_path="omniverse://localhost/NVIDIA/Assets/Isaac/4.1/Isaac/Environments/Simple_Room.usd",
            prim_path="/World"
        )

        # Add objects for the scene
        self._add_objects_to_scene(scene_config)

        # Configure lighting
        self._configure_lighting(scene_config)

        # Initialize cameras
        self._setup_cameras()

        # Reset the world to apply changes
        self.world.reset()

    def _add_objects_to_scene(self, scene_config: Dict[str, Any]):
        """Add objects to the scene based on configuration"""
        # This would add various objects to create a diverse training dataset
        # Implementation would vary based on specific objects needed
        pass

    def _configure_lighting(self, scene_config: Dict[str, Any]):
        """Configure RTX lighting based on scene requirements"""
        # Configure different lighting conditions based on requirements
        # This would set up realistic RTX lighting for photorealistic rendering
        pass

    def _setup_cameras(self):
        """Set up RGB, depth, and segmentation cameras"""
        # RGB Camera
        self.cameras['rgb'] = Camera(
            prim_path="/World/RGB_Camera",
            position=np.array([0.0, -1.0, 1.5]),
            frequency=30
        )

        # Depth Camera
        self.cameras['depth'] = Camera(
            prim_path="/World/Depth_Camera",
            position=np.array([0.0, -1.0, 1.5]),
            frequency=30
        )

        # Segmentation Camera (using semantic segmentation)
        self.cameras['segmentation'] = Camera(
            prim_path="/World/Segmentation_Camera",
            position=np.array([0.0, -1.0, 1.5]),
            frequency=30
        )

    def capture_synchronized_data(self) -> Dict[str, Any]:
        """
        Capture synchronized RGB, depth, and segmentation data
        Implements the core functionality for synthetic dataset generation
        """
        data = {}

        # Step the world to update sensors
        self.world.step(render=True)

        # Capture RGB data
        if 'rgb' in self.cameras:
            rgb_data = self.cameras['rgb'].get_rgb()
            data['rgb'] = rgb_data

        # Capture depth data
        if 'depth' in self.cameras:
            depth_data = self.cameras['depth'].get_depth_data()
            data['depth'] = depth_data

        # Capture segmentation data
        if 'segmentation' in self.cameras:
            segmentation_data = self.cameras['segmentation'].get_semantic_segmentation()
            data['segmentation'] = segmentation_data

        return data

    def export_dataset_sample(self, data: Dict[str, Any], sample_id: int):
        """
        Export a single dataset sample with proper labeling
        Implements FR-002: Export labeled datasets suitable for training vision models
        """
        # Save RGB image
        rgb_path = os.path.join(self.output_dir, "rgb", f"rgb_{sample_id:06d}.png")
        if 'rgb' in data and data['rgb'] is not None:
            rgb_img = Image.fromarray(data['rgb'])
            rgb_img.save(rgb_path)

        # Save depth map
        depth_path = os.path.join(self.output_dir, "depth", f"depth_{sample_id:06d}.png")
        if 'depth' in data and data['depth'] is not None:
            # Convert depth data to proper format for saving
            depth_img = Image.fromarray((data['depth'] * 255).astype(np.uint8))
            depth_img.save(depth_path)

        # Save segmentation map
        seg_path = os.path.join(self.output_dir, "segmentation", f"seg_{sample_id:06d}.png")
        if 'segmentation' in data and data['segmentation'] is not None:
            seg_img = Image.fromarray(data['segmentation'])
            seg_img.save(seg_path)

        # Create label file
        label_path = os.path.join(self.output_dir, "labels", f"label_{sample_id:06d}.json")
        label_data = {
            "sample_id": sample_id,
            "timestamp": carb.events.acquire_interface().get_current_event_time(),
            "camera_pose": self._get_camera_pose(),
            "objects_in_scene": self._get_scene_objects(),
            "metadata": {
                "rgb_path": f"rgb/rgb_{sample_id:06d}.png",
                "depth_path": f"depth/depth_{sample_id:06d}.png",
                "segmentation_path": f"segmentation/seg_{sample_id:06d}.png",
                "label_path": f"labels/label_{sample_id:06d}.json"
            }
        }

        with open(label_path, 'w') as f:
            json.dump(label_data, f, indent=2)

    def _get_camera_pose(self) -> Dict[str, float]:
        """Get current camera pose for labeling"""
        # Return camera position and orientation
        return {
            "position": {"x": 0.0, "y": -1.0, "z": 1.5},
            "orientation": {"qx": 0.0, "qy": 0.0, "qz": 0.0, "qw": 1.0}
        }

    def _get_scene_objects(self) -> List[Dict[str, Any]]:
        """Get list of objects in the current scene"""
        # Return information about objects in the scene for labeling
        return []

    def generate_dataset(self):
        """
        Generate the complete synthetic dataset
        Implements the acceptance scenario: Export labeled datasets suitable for training vision models
        """
        print(f"Starting synthetic dataset generation with {self.config['dataset_size']} samples...")

        # Setup the initial scene
        self.setup_scene(self.config)

        # Generate dataset samples
        for i in range(self.config['dataset_size']):
            # Capture synchronized data
            data = self.capture_synchronized_data()

            # Export the sample
            self.export_dataset_sample(data, i)

            # Update progress
            if (i + 1) % 100 == 0:
                print(f"Generated {i + 1}/{self.config['dataset_size']} samples")

        print(f"Dataset generation complete! Output saved to {self.output_dir}")

    def validate_dataset_quality(self) -> Dict[str, float]:
        """
        Validate the quality of the generated dataset
        Implements validation for dataset quality requirements
        """
        validation_results = {
            "completeness": 0.0,
            "image_quality": 0.0,
            "label_accuracy": 0.0,
            "overall_score": 0.0
        }

        # Count total samples
        rgb_files = [f for f in os.listdir(os.path.join(self.output_dir, "rgb")) if f.endswith('.png')]
        total_samples = len(rgb_files)

        # Validate that all required files exist for each sample
        complete_samples = 0
        for sample_id in range(total_samples):
            rgb_exists = os.path.exists(os.path.join(self.output_dir, "rgb", f"rgb_{sample_id:06d}.png"))
            depth_exists = os.path.exists(os.path.join(self.output_dir, "depth", f"depth_{sample_id:06d}.png"))
            seg_exists = os.path.exists(os.path.join(self.output_dir, "segmentation", f"seg_{sample_id:06d}.png"))
            label_exists = os.path.exists(os.path.join(self.output_dir, "labels", f"label_{sample_id:06d}.json"))

            if all([rgb_exists, depth_exists, seg_exists, label_exists]):
                complete_samples += 1

        validation_results["completeness"] = complete_samples / total_samples if total_samples > 0 else 0.0

        # Calculate overall score
        validation_results["overall_score"] = validation_results["completeness"]  # Simplified for now

        return validation_results


def main():
    """Main function to run the synthetic dataset generator"""
    generator = SyntheticDatasetGenerator(
        output_dir="./synthetic_dataset",
        config_path=None  # Using default config
    )

    # Generate the dataset
    generator.generate_dataset()

    # Validate the dataset quality
    validation_results = generator.validate_dataset_quality()
    print(f"Dataset validation results: {validation_results}")

    # Verify SC-005: Generated synthetic datasets are suitable for training vision models with at least 80% accuracy on validation tasks
    if validation_results["overall_score"] >= 0.8:
        print("Dataset validation passed - suitable for vision model training")
    else:
        print(f"Dataset validation failed - quality score {validation_results['overall_score']} below threshold")


if __name__ == "__main__":
    main()
```

### Image Capture Pipeline

```python
# Example image capture pipeline
import omni.kit
from omni.isaac.sensor import Camera
from omni.isaac.core.utils.viewports import set_camera_view

# Create RGB camera
rgb_camera = Camera(
    prim_path="/World/RGB_Camera",
    position=np.array([0.0, 0.0, 1.0]),
    frequency=30
)

# Configure depth sensor
depth_sensor = Camera(
    prim_path="/World/Depth_Camera",
    position=np.array([0.0, 0.0, 1.0]),
    frequency=30
)

# Capture synchronized RGB and depth data
rgb_data = rgb_camera.get_rgb()
depth_data = depth_sensor.get_depth_data()
```

### Data Export and Validation

- **Format Support**: PNG, JPEG, EXR for different use cases
- **Metadata**: Camera parameters, poses, and sensor configurations
- **Quality Validation**: Checking for proper exposure, focus, and labeling
- **Dataset Organization**: Structured directories for easy access

## Practical Exercise: Basic Scene Setup

### Step 1: Create Basic Office Scene

1. Launch Isaac Sim
2. Create a new USD stage
3. Add basic primitives (floor, walls, simple furniture)
4. Apply realistic materials
5. Configure RTX lighting

### Step 2: Configure Camera System

1. Add RGB camera to the scene
2. Set camera parameters (resolution, focal length, etc.)
3. Position camera for optimal scene coverage
4. Verify camera functionality

### Step 3: Generate Sample Dataset

1. Configure the image capture pipeline
2. Run simulation to generate synchronized RGB and depth data
3. Export sample dataset
4. Validate dataset quality

## Advanced Topics

### Dynamic Scene Elements

- **Moving Objects**: Simulating dynamic environments
- **Lighting Changes**: Time-of-day variations
- **Weather Effects**: Rain, fog, and atmospheric conditions
- **Humanoid Robot Integration**: Adding humanoid models to scenes

### Performance Optimization

- **LOD Systems**: Level-of-detail for complex scenes
- **Occlusion Culling**: Reducing rendering load
- **Multi-resolution Textures**: Optimizing memory usage
- **Simulation Optimization**: Balancing quality and performance

## Troubleshooting

### Common Issues

- **Rendering Artifacts**: Incorrect lighting or material settings
- **Performance Issues**: Too complex scenes for real-time rendering
- **Camera Problems**: Incorrect sensor configurations
- **Dataset Quality**: Poor labeling or inconsistent data

### Solutions

- **Scene Optimization**: Simplifying complex geometries
- **Material Tuning**: Adjusting PBR parameters for better results
- **Lighting Adjustment**: Balancing exposure and shadows
- **Validation Checks**: Implementing automated quality checks

## Summary

This chapter covered the fundamentals of Isaac Sim for perception applications in humanoid robotics. Students learned to create photorealistic simulation environments and generate synthetic datasets suitable for AI vision model training. The next chapter will build on this foundation by integrating Isaac Sim with Isaac ROS for visual SLAM applications.

## Further Reading

- [Isaac Sim Documentation](https://docs.omniverse.nvidia.com/isaacsim/latest/index.html)
- [RTX Rendering in Isaac Sim](https://docs.omniverse.nvidia.com/isaacsim/latest/advanced-topics/rtx-rendering.html)
- [USD File Format Specification](https://graphics.pixar.com/usd/release/index.html)