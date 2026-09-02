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

# 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Team Members](#-team-members)
- [Project Goals](#-project-goals)
- [Key Accomplishments](#-key-accomplishments)
- [System Architecture](#-system-architecture)
- [Project Demonstrations](#-project-demonstrations)
- [YOLO STOP-Sign Detection](#-yolo-stop-sign-detection)
- [Autonomous Lane Following](#-autonomous-lane-following)
- [STOP and Resume Behavior](#-stop-and-resume-behavior)
- [Conditional Pit-Stop Logic](#-conditional-pit-stop-logic)
- [Battery Monitoring](#-battery-monitoring)
- [Hardware](#-hardware)
- [Software](#-software)
- [ROS2 Topics](#-ros2-topics)
- [Repository Structure](#-repository-structure)
- [Challenges and Solutions](#-challenges-and-solutions)
- [Final Results](#-final-results)
- [Final Presentation](#-final-presentation)
- [Acknowledgments](#-acknowledgments)

---

# 🚀 Project Overview

This project implements a **condition-based autonomous pit-stop system** on the UCSD RoboCar platform using ROS2.

The RoboCar autonomously follows a marked track using an **OAK-D Lite camera**, detects a designated **STOP sign using a custom YOLO model**, monitors battery condition through the **VESC**, and determines whether the vehicle should continue driving or enter the pit.

When the STOP sign is detected:

- 🟢 **Battery healthy:** the RoboCar continues autonomous driving.
- 🔴 **Battery low:** the RoboCar initiates the autonomous pit-stop sequence.
- 🏁 The vehicle enters the pit area, stops for service, exits the pit, reacquires the lane, and resumes autonomous driving.

The project combines:

- Autonomous vehicle control
- ROS2 communication
- Computer vision
- Machine learning
- Real-time battery telemetry
- State-based decision making

---

# 👥 Team Members

| Team Member | Major | Project Role |
|---|---|---|
| **[Anthony Gaeta]** | [MAE] 
| **[Nicholas Campos]** | [ECE] 
| **[Qihao Huang]** | [ECE] 

### Team 5 — UC San Diego MAE 148

---

# 🎯 Project Goals

The objective of this project was to create a RoboCar capable of making an autonomous pit-stop decision based on both **environmental perception** and **vehicle condition**.

### Core Objectives

- ✅ Autonomous lane following
- ✅ Real-time STOP-sign detection
- ✅ Custom YOLO model deployment
- ✅ Battery-voltage monitoring
- ✅ Low-battery detection
- ✅ Conditional decision making
- ✅ STOP-sign response
- ✅ Autonomous pit entry
- ✅ Autonomous pit stop
- ✅ Pit exit
- ✅ Lane reacquisition
- ✅ Resume autonomous driving

---

# 🏆 Key Accomplishments

Team 5 integrated the major perception, decision, and control systems required for a conditional autonomous pit stop.

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

# 🧠 System Architecture

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

---

# 🎥 Project Demonstrations

This section documents the major stages of the completed autonomous system.

---

## 🤖 RoboCar Walkaround

<!-- DRAG ROBOT PHOTO HERE -->

![RoboCar Hardware](PASTE_ROBOT_HARDWARE_IMAGE_HERE)

### **RoboCar Hardware Demonstration**

▶️ **[WATCH ROBOT WALKAROUND VIDEO](PASTE_VIDEO_LINK_HERE)**

---

# 🛑 YOLO STOP-Sign Detection

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

<!-- DRAG A SCREENSHOT SHOWING YOUR YOLO BOUNDING BOX HERE -->

![YOLO STOP Detection](PASTE_YOLO_SCREENSHOT_HERE)

### **YOLO Detection Demonstration**

▶️ **[WATCH YOLO STOP-SIGN DETECTION DEMO](PASTE_YOLO_VIDEO_LINK_HERE)**

---

# 🛣️ Autonomous Lane Following

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

## Lane Detection Example

<!-- DRAG SCREENSHOT OF GREEN BOXES / LANE CALIBRATION HERE -->

![Lane Detection](PASTE_LANE_DETECTION_IMAGE_HERE)

## Track Driving

<!-- DRAG PHOTO OF CAR ON TRACK HERE -->

![RoboCar Track](PASTE_TRACK_IMAGE_HERE)

### **Autonomous Lane-Following Demonstration**

▶️ **[WATCH AUTONOMOUS TRACK DRIVING VIDEO](PASTE_LANE_VIDEO_LINK_HERE)**

---

# ⛔ STOP and Resume Behavior

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

## STOP Demonstration

<!-- DRAG SCREENSHOT OR PHOTO OF THE CAR STOPPED HERE -->

![STOP Demonstration](PASTE_STOP_IMAGE_HERE)

### **STOP → 4 Seconds → Resume Demo**

▶️ **[WATCH STOP AND RESUME DEMONSTRATION](PASTE_STOP_VIDEO_LINK_HERE)**

---

# 🔋 Conditional Pit-Stop Logic

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

## 🟢 Healthy Battery Scenario

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

▶️ **[WATCH HEALTHY BATTERY DEMONSTRATION](PASTE_VIDEO_LINK_HERE)**

---

## 🔴 Low Battery Scenario

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

---

# 🏁 Final Conditional Pit-Stop Demonstration

<!-- PUT YOUR BEST PIT STOP IMAGE HERE -->

![Final Pit Stop](PASTE_FINAL_PIT_STOP_IMAGE_HERE)

## **FINAL FULL-SYSTEM DEMONSTRATION**

### ▶️ **[WATCH THE COMPLETE AUTONOMOUS PIT-STOP DEMO](PASTE_FINAL_VIDEO_LINK_HERE)**

The demonstration shows the completed sequence:

**Lane Follow → STOP Detection → Battery Check → Pit Decision → Pit Entry → Stop → Pit Exit → Lane Rejoin → Resume Autonomous Driving**

---

# 🔋 Battery Monitoring

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

# 🔧 Hardware

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

## Hardware Layout

<!-- DRAG YOUR HARDWARE/WIRING PHOTO HERE -->

![Hardware Layout](PASTE_HARDWARE_IMAGE_HERE)

---

# 💻 Software

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

# 📡 ROS2 Topics

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

# 📂 Repository Structure

```text
UCSD-MAE148-SUMMER-II-Final_Project-Team_5/
│
├── src/
│   ├── stop_sign_node.py
│   ├── stop_manager_node.py
│   ├── battery_monitor_node.py
│   └── pit_stop_manager_node.py
│
├── models/
│   ├── best.onnx
│   └── README.md
│
├── config/
│   └── ros_racer_calibration.yaml
│
├── media/
│   ├── robot/
│   ├── yolo/
│   ├── lane/
│   ├── pitstop/
│   └── demos/
│
├── docs/
│
├── README.md
├── LICENSE
└── .gitignore
```

---

# 🧪 Challenges and Solutions

## YOLO Performance

Running neural-network inference on the Raspberry Pi while simultaneously performing autonomous lane following created computational-load concerns.

### Solution

YOLO inference was isolated in its own ROS2 node and throttled to approximately **4 Hz**, allowing the lane-following system to continue operating at a higher rate.

---

## STOP-Sign Confidence

A high confidence threshold worked during close-range testing but was less reliable when the physical STOP sign was viewed from realistic track distances and angles.

### Solution

The deployed threshold was tuned to approximately **0.35** while retaining a requirement for **three consecutive detections** to reduce false triggers.

---

## VESC Battery Telemetry

Initially, battery monitoring and motor control attempted to access the VESC serial connection independently.

### Solution

The VESC drive node became the single owner of the serial device. Battery voltage is now published through ROS2 and processed by a separate battery-monitor node.

---

## Command Routing

The lane-guidance node originally published directly to `/cmd_vel`, which would bypass the STOP/pit logic.

### Solution

Lane guidance was remapped to:

```text
/lane_cmd_vel
```

The pit-stop manager receives the lane command and becomes the only final publisher to:

```text
/cmd_vel
```

---

## Hardware Connectivity

USB and VESC serial communication required careful troubleshooting during integration.

The final system verified the VESC serial connection, camera connection, ROS2 topic flow, and motor response before autonomous track testing.

---

# ✅ Final Results

The completed system demonstrates integration of:

- ✅ Autonomous lane detection
- ✅ Autonomous lane following
- ✅ YOLO STOP-sign recognition
- ✅ Multi-frame detection confirmation
- ✅ STOP and resume control
- ✅ Real-time battery telemetry
- ✅ Battery percentage estimation
- ✅ Low-battery classification
- ✅ Conditional autonomous decision making
- ✅ Autonomous pit entry
- ✅ Autonomous pit stop
- ✅ Autonomous pit exit
- ✅ Lane reacquisition
- ✅ Return to normal autonomous driving

---

# 📊 Final Presentation

## Team 5 Final Presentation

📄 **[VIEW FINAL PRESENTATION](PASTE_PRESENTATION_LINK_HERE)**

---

# 🎬 Demo Videos

| Demonstration | Link |
|---|---|
| 🤖 RoboCar Walkaround | **[Watch](PASTE_LINK)** |
| 🛑 YOLO STOP Detection | **[Watch](PASTE_LINK)** |
| 🛣️ Autonomous Lane Following | **[Watch](PASTE_LINK)** |
| ⛔ STOP → 4 sec → Resume | **[Watch](PASTE_LINK)** |
| 🔋 Battery Monitoring | **[Watch](PASTE_LINK)** |
| 🏁 Final Conditional Pit Stop | **[Watch](PASTE_LINK)** |

---

# 🙏 Acknowledgments

We would like to thank the **UCSD MAE 148 instructional team** for providing the RoboCar platform, course infrastructure, hardware resources, and technical guidance used throughout this project.

---

<div align="center">

# Team 5

### UC San Diego

### MAE 148 — Introduction to Autonomous Vehicles

### Summer Session II 2026

**Conditional Autonomous Pit Stop**

</div>
