---
sidebar_position: 8
---

# Ch 8 — Unity High-Fidelity Digital Twin

## Introduction to Unity for Robotics Visualization

Unity provides the high-fidelity visualization layer of the digital twin, offering photorealistic rendering, immersive user interfaces, and intuitive human-robot interaction capabilities. Unlike Gazebo's physics-focused approach, Unity excels at creating visually compelling representations that enable operators to understand robot behavior and environment interactions in an intuitive, human-centered way.

Key advantages of Unity in the digital twin architecture:
- **Photorealistic Rendering**: High-quality graphics for immersive visualization
- **Intuitive Interfaces**: User-friendly controls and monitoring systems
- **Virtual Reality Support**: VR capabilities for immersive robot operation
- **Real-time Performance**: Smooth, high-frame-rate rendering for responsive interaction

## Importing URDF/FBX Models to Unity

### Using Unity URDF Importer
The Unity URDF Importer package provides seamless integration between ROS 2 robot descriptions and Unity:

```csharp
// RobotStateSubscriber.cs - Example Unity script to receive robot state
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using RosMessageTypes.Sensor;
using RosMessageTypes.Control;

public class RobotStateSubscriber : MonoBehaviour
{
    [Header("ROS Connection")]
    public string rosBridgeUrl = "ws://localhost:9090";

    [Header("Robot Configuration")]
    public GameObject robotModel;  // The imported robot model
    public string[] jointNames;    // Joint names matching URDF
    public Transform[] jointObjects; // Corresponding transforms in Unity

    private Dictionary<string, float> jointPositions = new Dictionary<string, float>();

    void Start()
    {
        // Initialize ROS connection
        InitializeRosConnection();
    }

    private void InitializeRosConnection()
    {
        // Setup ROS connection using Unity Robotics packages
        // This would connect to the ROS bridge server
    }

    public void UpdateJointPositions(JointStateMsg jointState)
    {
        // Update joint positions from ROS message
        for (int i = 0; i < jointState.name.Length; i++)
        {
            string jointName = jointState.name[i];
            float position = (float)jointState.position[i];

            if (jointPositions.ContainsKey(jointName))
            {
                jointPositions[jointName] = position;
            }
        }

        // Apply positions to Unity transforms
        ApplyJointTransforms();
    }

    private void ApplyJointTransforms()
    {
        for (int i = 0; i < jointNames.Length; i++)
        {
            string jointName = jointNames[i];
            if (jointPositions.ContainsKey(jointName))
            {
                // Apply the joint position to the corresponding Unity transform
                // This depends on the joint type (revolute, prismatic, etc.)
                ApplyJointTransform(jointObjects[i], jointPositions[jointName]);
            }
        }
    }

    private void ApplyJointTransform(Transform jointTransform, float position)
    {
        // For revolute joints, apply rotation
        // For prismatic joints, apply translation
        // This is a simplified example
        jointTransform.localRotation = Quaternion.Euler(0, position * Mathf.Rad2Deg, 0);
    }
}
```

### Model Scaling and Coordinate System Conversion
Proper scaling and coordinate system alignment are crucial for accurate visualization:

- **Units**: Convert from meters (ROS/Gazebo) to Unity's system
- **Coordinate System**: Handle differences between ROS (x-forward, y-left, z-up) and Unity (x-right, y-up, z-forward)
- **Scale Factors**: Apply appropriate scaling for visual consistency

## Realistic Lighting and Shaders

### Physically-Based Rendering (PBR)
Unity's PBR materials provide realistic surface interactions:

```csharp
// MaterialSetup.cs - Configure realistic materials for robot
using UnityEngine;

public class MaterialSetup : MonoBehaviour
{
    [Header("Material Properties")]
    public float metallic = 0.5f;
    public float smoothness = 0.7f;
    public Color baseColor = Color.gray;

    void Start()
    {
        SetupRobotMaterials();
    }

    void SetupRobotMaterials()
    {
        Renderer[] renderers = GetComponentsInChildren<Renderer>();

        foreach (Renderer renderer in renderers)
        {
            Material material = renderer.material;
            material.SetFloat("_Metallic", metallic);
            material.SetFloat("_Smoothness", smoothness);
            material.SetColor("_Color", baseColor);

            // Add wear and tear textures for realistic appearance
            AddRealisticTextures(material);
        }
    }

    void AddRealisticTextures(Material material)
    {
        // Add normal maps, roughness maps, etc. for realistic appearance
        // This could include scratches, wear patterns, or manufacturing details
    }
}
```

### Dynamic Lighting
Implement dynamic lighting to enhance the visual quality:

```csharp
// DynamicLighting.cs - Real-time lighting effects
using UnityEngine;

public class DynamicLighting : MonoBehaviour
{
    [Header("Lighting Configuration")]
    public Light mainLight;
    public float intensityVariation = 0.1f;
    public float flickerSpeed = 2.0f;

    void Update()
    {
        // Add subtle light variation for realism
        float variation = Mathf.Sin(Time.time * flickerSpeed) * intensityVariation;
        mainLight.intensity = 1.0f + variation;
    }
}
```

## Human-Robot Interaction Scene

### Creating Interactive Interfaces
Design intuitive interfaces for human-robot interaction:

```csharp
// RobotControlPanel.cs - Interactive control panel
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class RobotControlPanel : MonoBehaviour
{
    [Header("UI Elements")]
    public Button moveForwardButton;
    public Button moveBackwardButton;
    public Button turnLeftButton;
    public Button turnRightButton;
    public Slider speedSlider;
    public TextMeshProUGUI statusText;

    [Header("Robot Communication")]
    public RobotStateSubscriber robotSubscriber;

    void Start()
    {
        SetupButtonEvents();
    }

    void SetupButtonEvents()
    {
        moveForwardButton.onClick.AddListener(() => SendCommand("move_forward"));
        moveBackwardButton.onClick.AddListener(() => SendCommand("move_backward"));
        turnLeftButton.onClick.AddListener(() => SendCommand("turn_left"));
        turnRightButton.onClick.AddListener(() => SendCommand("turn_right"));

        speedSlider.onValueChanged.AddListener(OnSpeedChanged);
    }

    void SendCommand(string command)
    {
        // Send command to robot via ROS bridge
        Debug.Log($"Sending command: {command}");

        // Update status
        statusText.text = $"Command: {command}";
    }

    void OnSpeedChanged(float speed)
    {
        Debug.Log($"Speed changed to: {speed}");
        // Send speed parameter to robot
    }
}
```

### Visualization of Sensor Data
Display sensor data in an intuitive, visual format:

```csharp
// SensorVisualization.cs - Visualize sensor data in Unity
using UnityEngine;

public class SensorVisualization : MonoBehaviour
{
    [Header("Sensor Data Sources")]
    public RobotStateSubscriber robotState;
    public GameObject lidarVisualization;
    public GameObject depthVisualization;
    public GameObject imuVisualization;

    [Header("Visualization Settings")]
    public Color obstacleColor = Color.red;
    public Color safeColor = Color.green;
    public float visualizationRadius = 0.1f;

    void Update()
    {
        UpdateSensorVisualizations();
    }

    void UpdateSensorVisualizations()
    {
        // Visualize LiDAR data as point cloud or obstacle indicators
        VisualizeLidarData();

        // Visualize depth camera data as point cloud
        VisualizeDepthData();

        // Visualize IMU data as orientation indicator
        VisualizeImuData();
    }

    void VisualizeLidarData()
    {
        // Create visual indicators for LiDAR obstacles
        // This could be point clouds, obstacle spheres, or distance rings
    }

    void VisualizeDepthData()
    {
        // Create 3D visualization of depth camera point cloud
        // Could overlay on the 3D scene or show in separate view
    }

    void VisualizeImuData()
    {
        // Show orientation and balance status
        // Could be as a 3D orientation widget or balance indicator
    }
}
```

## Unity-ROS 2 Bridge Connection

### Setting up the Bridge
The Unity-ROS bridge enables real-time communication between Unity and ROS 2 systems:

```csharp
// ROSBridgeConnection.cs - Main bridge connection
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.Robotics.ROSTCPConnector;
using RosMessageTypes.Sensor;
using RosMessageTypes.Navigation;

public class ROSBridgeConnection : MonoBehaviour
{
    [Header("Connection Settings")]
    public string rosBridgeIP = "127.0.0.1";
    public int rosBridgePort = 9090;

    [Header("ROS Topics")]
    public string jointStateTopic = "/joint_states";
    public string lidarTopic = "/lidar/scan";
    public string imuTopic = "/imu/data";
    public string cmdVelTopic = "/cmd_vel";

    private ROSConnection ros;
    private RobotStateSubscriber robotState;

    void Start()
    {
        // Initialize ROS connection
        ros = ROSConnection.instance;
        ros.Initialize(rosBridgeIP, rosBridgePort);

        // Subscribe to topics
        ros.Subscribe<JointStateMsg>(jointStateTopic, OnJointStateReceived);
        ros.Subscribe<LaserScanMsg>(lidarTopic, OnLidarReceived);
        ros.Subscribe<ImuMsg>(imuTopic, OnImuReceived);

        robotState = GetComponent<RobotStateSubscriber>();
    }

    void OnJointStateReceived(JointStateMsg jointState)
    {
        // Update robot visualization with new joint states
        robotState.UpdateJointPositions(jointState);
    }

    void OnLidarReceived(LaserScanMsg lidarData)
    {
        // Process LiDAR data for visualization
        ProcessLidarData(lidarData);
    }

    void OnImuReceived(ImuMsg imuData)
    {
        // Process IMU data for balance visualization
        ProcessImuData(imuData);
    }

    void ProcessLidarData(LaserScanMsg lidarData)
    {
        // Convert LiDAR data to Unity visualization format
        // Could create point cloud or obstacle indicators
    }

    void ProcessImuData(ImuMsg imuData)
    {
        // Use IMU data for balance indicators or orientation visualization
    }

    public void SendVelocityCommand(float linear, float angular)
    {
        // Send velocity command to robot
        var cmd = new TwistMsg();
        cmd.linear = new Vector3Msg(linear, 0, 0);
        cmd.angular = new Vector3Msg(0, 0, angular);

        ros.Publish(cmdVelTopic, cmd);
    }
}
```

### Synchronization Strategies
Maintain synchronization between Gazebo physics and Unity visualization:

```csharp
// SynchronizationManager.cs - Keep Unity in sync with Gazebo
using UnityEngine;

public class SynchronizationManager : MonoBehaviour
{
    [Header("Synchronization Settings")]
    public float maxLatency = 0.1f;  // Maximum acceptable latency in seconds
    public float syncThreshold = 0.05f;  // Threshold for correction

    [Header("Timing")]
    public float physicsUpdateRate = 100.0f;  // Expected physics rate in Hz
    public float visualizationUpdateRate = 60.0f;  // Visualization rate in Hz

    private float lastSyncTime;
    private float targetDeltaTime;
    private float visualizationDeltaTime;

    void Start()
    {
        lastSyncTime = Time.time;
        targetDeltaTime = 1.0f / physicsUpdateRate;
        visualizationDeltaTime = 1.0f / visualizationUpdateRate;
    }

    void Update()
    {
        CheckSynchronization();
    }

    void CheckSynchronization()
    {
        float currentTime = Time.time;
        float elapsed = currentTime - lastSyncTime;

        // Check if we're falling behind
        if (elapsed > maxLatency)
        {
            Debug.LogWarning("Unity visualization is lagging behind physics simulation");
            // Implement catch-up strategy if needed
        }

        // Check for excessive synchronization error
        float expectedElapsed = targetDeltaTime;
        float syncError = Mathf.Abs(elapsed - expectedElapsed);

        if (syncError > syncThreshold)
        {
            Debug.Log($"Synchronization error detected: {syncError:F3}s");
            // Log for analysis - actual correction depends on specific requirements
        }

        lastSyncTime = currentTime;
    }

    public void ApplyPhysicsCorrection(Vector3 position, Quaternion rotation)
    {
        // Apply correction to Unity objects based on physics state
        transform.position = position;
        transform.rotation = rotation;
    }
}
```

## Performance Optimization

### Frame Rate Management
Maintain 30+ FPS for smooth visualization:

- **Level of Detail (LOD)**: Use simplified models when far from camera
- **Occlusion Culling**: Don't render objects not visible to camera
- **Texture Streaming**: Load textures at appropriate resolution
- **Shader Optimization**: Use efficient shaders for real-time rendering

### Resource Management
Efficiently manage Unity resources for sustained performance:

```csharp
// ResourceManager.cs - Efficient resource management
using UnityEngine;
using System.Collections.Generic;

public class ResourceManager : MonoBehaviour
{
    [Header("Resource Pools")]
    public int obstacleVisualizationPoolSize = 100;
    public int pointCloudPointPoolSize = 1000;

    private Queue<GameObject> obstaclePool;
    private Queue<GameObject> pointPool;

    void Start()
    {
        InitializePools();
    }

    void InitializePools()
    {
        obstaclePool = new Queue<GameObject>();
        pointPool = new Queue<GameObject>();

        // Pre-instantiate objects for performance
        for (int i = 0; i < obstacleVisualizationPoolSize; i++)
        {
            GameObject obj = CreateObstacleVisualization();
            obj.SetActive(false);
            obstaclePool.Enqueue(obj);
        }

        for (int i = 0; i < pointCloudPointPoolSize; i++)
        {
            GameObject obj = CreatePointCloudPoint();
            obj.SetActive(false);
            pointPool.Enqueue(obj);
        }
    }

    GameObject GetObstacleVisualization()
    {
        if (obstaclePool.Count > 0)
        {
            GameObject obj = obstaclePool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        else
        {
            // Create new if pool is empty
            return CreateObstacleVisualization();
        }
    }

    void ReturnObstacleVisualization(GameObject obj)
    {
        obj.SetActive(false);
        obstaclePool.Enqueue(obj);
    }

    GameObject CreateObstacleVisualization()
    {
        GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        obj.transform.SetParent(transform);
        obj.GetComponent<Renderer>().material.color = Color.red;
        return obj;
    }

    GameObject CreatePointCloudPoint()
    {
        GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        obj.transform.SetParent(transform);
        obj.transform.localScale = Vector3.one * 0.01f;  // Small point
        obj.GetComponent<Renderer>().material.color = Color.blue;
        return obj;
    }
}
```

## Integration Testing

### Bridge Stability Testing
Verify the Unity-ROS bridge operates stably:

- **Connection Persistence**: Test long-running connections (10+ minutes)
- **Data Throughput**: Verify sensor data flows without loss
- **Latency Monitoring**: Ensure &lt;100ms latency between Gazebo and Unity

### Cross-Platform Compatibility
Test the digital twin on target platforms:

- **Windows**: Primary development platform
- **Linux**: ROS 2 compatibility
- **VR Platforms**: If VR support is required

## Practical Implementation Example

Here's a complete example of a Unity scene that visualizes a humanoid robot:

```csharp
// DigitalTwinManager.cs - Complete digital twin manager
using UnityEngine;
using Unity.Robotics.ROSTCPConnector;
using RosMessageTypes.Sensor;
using RosMessageTypes.Control;

public class DigitalTwinManager : MonoBehaviour
{
    [Header("Robot Visualization")]
    public GameObject robotModel;
    public RobotStateSubscriber stateSubscriber;
    public SensorVisualization sensorVis;

    [Header("Performance")]
    public int targetFrameRate = 60;
    public TextMeshProUGUI performanceText;

    void Start()
    {
        // Set target frame rate
        Application.targetFrameRate = targetFrameRate;

        // Initialize components
        InitializeDigitalTwin();
    }

    void InitializeDigitalTwin()
    {
        // Setup ROS connection
        var ros = ROSConnection.instance;
        ros.Initialize("127.0.0.1", 9090);

        // Subscribe to robot topics
        ros.Subscribe<JointStateMsg>("/joint_states", OnJointState);
        ros.Subscribe<ImuMsg>("/imu/data", OnImuData);
        ros.Subscribe<LaserScanMsg>("/lidar/scan", OnLidarData);

        Debug.Log("Digital twin initialized successfully");
    }

    void OnJointState(JointStateMsg jointState)
    {
        stateSubscriber.UpdateJointPositions(jointState);
    }

    void OnImuData(ImuMsg imuData)
    {
        // Update balance visualization
        UpdateBalanceIndicator(imuData);
    }

    void OnLidarData(LaserScanMsg lidarData)
    {
        // Update obstacle visualization
        sensorVis.ProcessLidarData(lidarData);
    }

    void UpdateBalanceIndicator(ImuMsg imuData)
    {
        // Calculate and display balance status
        float roll = Mathf.Atan2(2 * (imuData.orientation.w * imuData.orientation.x +
                                    imuData.orientation.y * imuData.orientation.z),
                                1 - 2 * (imuData.orientation.x * imuData.orientation.x +
                                        imuData.orientation.y * imuData.orientation.y));

        float pitch = Mathf.Asin(2 * (imuData.orientation.w * imuData.orientation.y -
                                    imuData.orientation.z * imuData.orientation.x));

        // Visualize balance (e.g., color changes based on stability)
        if (Mathf.Abs(roll) > 0.2f || Mathf.Abs(pitch) > 0.2f)
        {
            // Robot is unstable
            robotModel.GetComponent<Renderer>().material.color = Color.red;
        }
        else
        {
            // Robot is stable
            robotModel.GetComponent<Renderer>().material.color = Color.green;
        }
    }

    void Update()
    {
        // Update performance metrics
        if (performanceText != null)
        {
            float fps = 1.0f / Time.unscaledDeltaTime;
            performanceText.text = $"FPS: {fps:F1}";
        }
    }
}
```

## Summary

This chapter covered the Unity high-fidelity digital twin implementation, including model import, realistic visualization, human-robot interaction interfaces, and ROS 2 bridge connectivity. We explored performance optimization techniques and practical implementation examples. The digital twin system now provides a complete pipeline from physics simulation in Gazebo to high-fidelity visualization in Unity, all connected through ROS 2 communication.

With all four chapters complete, the Digital Twin Simulation Module provides comprehensive coverage of digital twin fundamentals, Gazebo physics, sensor simulation, and Unity visualization for humanoid robotics.