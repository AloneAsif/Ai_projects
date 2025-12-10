---
sidebar_position: 2
---

# Ch 1 — ROS 2 Overview

## ROS 2 as Robotic Nervous System

ROS 2 (Robot Operating System 2) serves as the nervous system for robotic applications, providing a communication framework that enables different components of a robot to work together seamlessly. Just as the nervous system connects different parts of the human body, ROS 2 connects various sensors, actuators, controllers, and computational modules in a robot.

## Nodes, Topics, Services, and Actions

### Nodes
A node is a process that performs computation. In ROS 2, nodes are the fundamental building blocks of a robotic application. Each node typically performs a specific task such as sensor data processing, motion planning, or control. Nodes can be written in different programming languages (C++, Python, etc.) and can run on different machines within a distributed system.

### Topics
Topics provide a publish-subscribe communication pattern in ROS 2. Publishers send data to topics, and subscribers receive data from topics. This pattern enables asynchronous communication where publishers and subscribers don't need to know about each other directly. Topic communication is ideal for streaming data like sensor readings, camera feeds, or robot state information.

### Services
Services provide a request-response communication pattern in ROS 2. A client sends a request to a service, and the service processes the request and returns a response. This synchronous communication is ideal for operations that require a specific result, such as triggering a specific action, querying robot state, or performing a computation.

### Actions
Actions provide a goal-feedback-result communication pattern for long-running tasks. A client sends a goal to an action server, receives feedback during execution, and gets a final result. Actions are ideal for tasks that take time to complete, such as navigation to a specific location or manipulation of an object.

## Simple Graph Diagrams

### Basic ROS 2 Graph
```
[Sensor Node] ----(sensor_data)----> [Processing Node] ----(processed_data)----> [Controller Node]
       |                                    |                                           |
    Camera                        Object Detection                      Motor Commands
```

### Publisher-Subscriber Pattern
```
Publisher Node A
       |
       | (Topic: /sensor_data)
       |
[Message Bus (ROS 2 Topic)]
       |
       | (Topic: /sensor_data)
       |
       +--> Subscriber Node B
       |
       +--> Subscriber Node C
       |
       +--> Subscriber Node D
```

### Service Request-Response Pattern
```
Client Node                        Service Server Node
     |                                    |
     |--- Request (Service: /calculate)-->|
     |                                    |
     |<-- Response (Result) --------------|
```

### Action Goal-Feedback-Result Pattern
```
Action Client                      Action Server
     |                                    |
     |--- Goal (Move to Location)------->|
     |                                    |
     |<-- Feedback (Progress: 30%) -------|
     |<-- Feedback (Progress: 60%) -------|
     |                                    |
     |<-- Result (Success/Failure) -------|
```

## Key Concepts

- **ROS 2 Graph**: The network of nodes and their connections
- **Communication Patterns**: Different ways nodes can interact (topics, services, actions)
- **Distributed Architecture**: Nodes can run on different machines
- **Language Independence**: Nodes can be written in different programming languages
- **Package Management**: ROS 2 packages organize related functionality

## Summary

This chapter introduced the fundamental concepts of ROS 2 as a robotic nervous system. We explored the four main communication patterns: nodes, topics, services, and actions. In the next chapter, we'll get hands-on experience implementing these concepts using Python (rclpy).