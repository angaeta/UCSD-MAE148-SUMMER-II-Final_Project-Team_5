<div align="center">

#  Conditional Autonomous Pit Stop

## Team 5 — UCSD MAE 148

### Introduction to Autonomous Vehicles — Summer Session II 2026

**ROS2 • Computer Vision • YOLO • Autonomous Driving • Battery Monitoring**

</div>

---

# Final RoboCar

<!-- DRAG AND DROP YOUR BEST PHOTO OF THE FINISHED ROBOT BELOW -->

![Team 5 RoboCar](PASTE_ROBOT_IMAGE_HERE)

<div align="center">

**Team 5 Conditional Autonomous Pit-Stop RoboCar**

</div>

---

# Table of Contents

- [Final RoboCar](#final-robocar)
- [Project Overview](#project-overview)
- [Team Members](#team-members)
- [Project Goals](#project-goals)
- [Key Accomplishments](#key-accomplishments)
- [System Architecture](#system-architecture)
- [YOLO STOP-Sign Detection](#yolo-stop-sign-detection)
- [Autonomous Lane Following](#autonomous-lane-following)
- [STOP and Resume Behavior](#stop-and-resume-behavior)
- [Conditional Pit-Stop Logic](#conditional-pit-stop-logic)
  - [Healthy Battery Scenario](#healthy-battery-scenario)
  - [Low Battery Scenario](#low-battery-scenario)
- [Battery Monitoring](#battery-monitoring)
- [Hardware](#hardware)
- [Software](#software)
- [ROS2 Topics](#ros2-topics)

---

# Project Overview

This project implements a **condition-based autonomous pit-stop system** on the UCSD RoboCar platform using ROS2.

The RoboCar autonomously follows a marked track using an **OAK-D Lite camera**, detects a designated **STOP sign using a custom YOLO model**, monitors battery condition through the **VESC**, and determines whether the vehicle should continue driving or enter the pit.

When the STOP sign is detected:

-  **Battery healthy:** the RoboCar continues autonomous driving.
-  **Battery low:** the RoboCar initiates the autonomous pit-stop sequence.
-  The vehicle enters the pit area, stops for service, exits the pit, reacquires the lane, and resumes autonomous driving.

The project combines:

- Autonomous vehicle control
- ROS2 communication
- Computer vision
- Machine learning
- Real-time battery telemetry
- State-based decision making

---

#  Team Members

 **Anthony Gaeta**  [MAE] 
 **Nicholas Campos**  [ECE] 
 **Qihao Huang**  [ECE] 

### Team 5 — UC San Diego MAE 148

---

#  Project Goals

The objective of this project was to create a RoboCar capable of making an autonomous pit-stop decision based on both **environmental perception** and **vehicle condition**.

### Core Objectives

-  Autonomous lane following
-  Real-time STOP-sign detection
-  Custom YOLO model deployment
-  Battery-voltage monitoring
-  Low-battery detection
-  Conditional decision making
-  STOP-sign response
-  Autonomous pit entry
-  Autonomous pit stop
-  Pit exit
-  Lane reacquisition
-  Resume autonomous driving

---

#  Key Accomplishments

Integrated the major perception, decision, and control systems required for a conditional autonomous pit stop.

### Perception

- Integrated the OAK-D Lite camera with ROS2
- Calibrated yellow-lane detection
- Trained a custom YOLO model for STOP-sign detection
- Converted the model to ONNX for Raspberry Pi inference
- Implemented ONNX Runtime inference on the RoboCar
- Added multi-frame confirmation to reduce false detections

### Vehicle Control

- Implemented autonomous lane following
- Routed lane commands through a higher-level stop/pit manager
- Integrated VESC throttle and steering control
- Implemented safe zero-velocity stopping
- Implemented autonomous restart after stopping

### Battery Monitoring

- Added real-time VESC battery telemetry
- Published battery voltage through ROS2
- Estimated battery percentage
- Implemented low-battery detection
- Added hysteresis to prevent rapid switching near the threshold

### Autonomous Decision Making

- Combined STOP-sign detection with battery state
- Healthy battery → continue driving
- Low battery → request pit stop
- Enter pit
- Stop for service
- Exit pit
- Rejoin lane
- Resume normal autonomous driving

---

# System Architecture

```text
                         OAK-D Lite
                             |
                  +----------+----------+
                  |                     |
                  v                     v
           Lane Detection        YOLO STOP Detection
                  |                     |
                  v                     |
           Lane Guidance                |
                  |                     |
                  +----------+----------+
                             |
                      Pit Stop Manager
                             |
                     Battery Condition
                        /          \
                   Healthy         Low
                      |             |
               Continue Driving   PIT ENTRY
                                    |
                                 PIT STOP
                                    |
                                 PIT EXIT
                                    |
                                  REJOIN
                                    |
                           Autonomous Driving
                                    |
                                 /cmd_vel
                                    |
                                   VESC
                                    |
                          Motor + Steering
```

# YOLO STOP-Sign Detection

A custom YOLO object-detection model was trained to identify the physical STOP sign used as the pit-area marker.

The neural-network detector runs independently from the lane-detection pipeline so that YOLO inference does not significantly interfere with the autonomous steering loop.

## Detection Pipeline

```text
OAK-D RGB Camera
       |
       v
/camera/color/image_0
       |
       v
Custom YOLO Model
       |
       v
ONNX Runtime
       |
       v
Confidence Threshold
       |
       v
3 Consecutive Detections
       |
       v
/stop_sign_detected
```

## Model Configuration

| Parameter | Value |
|---|---|
| Model | Custom YOLO STOP-sign detector |
| Deployment Format | ONNX |
| Input Resolution | 640 × 640 |
| Confidence Threshold | 0.35 |
| Required Detections | 3 consecutive detections |
| Inference Rate | ~4 Hz |
| Camera Topic | `/camera/color/image_0` |
| Output Topic | `/stop_sign_detected` |

## YOLO Detection Example
<img width="1033" height="658" alt="Screenshot 2026-09-01 200038" src="https://github.com/user-attachments/assets/f247c93a-851a-40e4-87a4-a6ed44710c38" />



**[WATCH YOLO STOP-SIGN DETECTION DEMO](https://www.youtube.com/watch?v=bK1lqh1ho3Q)**

---

# Autonomous Lane Following

The OAK-D Lite provides RGB imagery for the lane-detection system.

The lane-detection node identifies the yellow track markers and calculates the lane position. The lane-guidance node then generates steering and throttle commands.

```text
/camera/color/image_0
          |
          v
 lane_detection_node
          |
      /centroid
          |
          v
 lane_guidance_node
          |
    /lane_cmd_vel
          |
          v
   Pit Stop Manager
          |
          v
       /cmd_vel
```

#  STOP and Resume Behavior

Before implementing the complete conditional pit-stop state machine, the STOP-sign behavior was validated independently.

When the vehicle detects the STOP sign:

```text
Autonomous Lane Following
          |
          v
STOP Sign Detected
          |
          v
Publish Zero Velocity
          |
          v
STOP
          |
      Wait 4 Seconds
          |
          v
Resume Lane Following
```

The vehicle successfully demonstrated:

### **DRIVE → DETECT STOP → STOP → WAIT 4 SECONDS → RESUME**

The stop manager acts as the final publisher of vehicle commands and prevents the lane-guidance node from bypassing the stopping logic.

### **STOP → 4 Seconds → Resume Demo**

▶️ **[WATCH STOP AND RESUME DEMONSTRATION](https://youtu.be/hoZTC7xIytY)**

---

#  Conditional Pit-Stop Logic

The final project extends the STOP-sign behavior by adding battery condition to the decision.

```text
                    NORMAL DRIVING
                          |
                          v
                  STOP SIGN DETECTED
                          |
                          v
                    CHECK BATTERY
                     /          \
                    /            \
           BATTERY HEALTHY     BATTERY LOW
                  |                 |
                  v                 v
          CONTINUE DRIVING      PIT REQUEST
                                    |
                                    v
                                PIT ENTRY
                                    |
                                    v
                                PIT STOP
                                    |
                                    v
                                PIT EXIT
                                    |
                                    v
                                  REJOIN
                                    |
                                    v
                              NORMAL DRIVING
```

---

## Healthy Battery Scenario

When:

```text
STOP Sign = Detected
Battery Low = False
```

The RoboCar does **not** enter the pit and continues autonomous driving.

```text
STOP Detected
      +
Battery Healthy
      |
      v
Continue Driving
```

### Healthy-Battery Demo

▶️ **[WATCH HEALTHY BATTERY DEMONSTRATION](https://youtu.be/omp8wu0ba2c)**

---

## Low Battery Scenario

When:

```text
STOP Sign = Detected
Battery Low = True
```

the RoboCar initiates the pit-stop sequence.

```text
STOP Detected
      +
Battery Low
      |
      v
PIT REQUEST
      |
      v
PIT ENTRY
      |
      v
PIT STOP
      |
      v
PIT EXIT
      |
      v
REJOIN
```


# Battery Monitoring

The vehicle uses the VESC to provide real-time battery-voltage telemetry while retaining motor and steering control.

The VESC drive node owns the serial connection and publishes:

```text
/battery_voltage
```

The Team 5 battery-monitor node processes this voltage and publishes:

```text
/battery_percentage
/battery_low
```

## Battery Parameters

| Parameter | Value |
|---|---:|
| Battery Type | 4S LiPo |
| Capacity | 3300 mAh |
| Nominal Voltage | 14.8 V |
| Full Voltage | ~16.8 V |
| Low Battery Threshold | 15.0 V |
| Recovery Threshold | 15.2 V |
| Moving Average | 5 samples |

Hysteresis prevents the battery state from rapidly changing between LOW and HEALTHY when the measured voltage is near the threshold.

## Battery Data Flow

```text
VESC
 |
 v
Raw Battery Voltage
 |
 v
/battery_voltage
 |
 v
battery_monitor_node
 |
 +-------------------+
 |                   |
 v                   v
/battery_percentage  /battery_low
```

---

#  Hardware

## RoboCar Components

- Raspberry Pi
- OAK-D Lite camera
- VESC motor controller
- Brushless DC drive motor
- Steering servo
- 4S LiPo battery
- USB hub
- Wi-Fi adapter
- UCSD RoboCar chassis

---

# Software

The project was developed using:

| Technology | Purpose |
|---|---|
| **ROS2 Jazzy** | Robot communication and control |
| **Python 3** | ROS2 nodes and vehicle logic |
| **OpenCV** | Image processing |
| **YOLO** | STOP-sign object detection |
| **ONNX Runtime** | Raspberry Pi model inference |
| **DepthAI** | OAK-D camera interface |
| **CvBridge** | ROS image conversion |
| **PyVESC** | Motor controller communication |
| **NumPy** | Model preprocessing and numerical operations |

---

# ROS2 Topics

| Topic | Message Type | Purpose |
|---|---|---|
| `/camera/color/image_0` | `sensor_msgs/msg/Image` | OAK-D RGB camera |
| `/centroid` | Lane information | Detected lane position |
| `/lane_cmd_vel` | `geometry_msgs/msg/Twist` | Lane-following command |
| `/stop_sign_detected` | `std_msgs/msg/Bool` | YOLO STOP detection |
| `/battery_voltage` | `std_msgs/msg/Float32` | Raw VESC voltage |
| `/battery_percentage` | `std_msgs/msg/Float32` | Estimated state of charge |
| `/battery_low` | `std_msgs/msg/Bool` | Battery condition |
| `/cmd_vel` | `geometry_msgs/msg/Twist` | Final motor/steering command |

---

