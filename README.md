# Conditional Autonomous Pit Stop — Team 5

## Project Overview

This project implements a condition-based autonomous pit-stop system on the UCSD RoboCar platform using ROS2.

The vehicle performs autonomous lane following using an OAK-D Lite camera, detects a designated STOP sign using a custom YOLO model, monitors battery voltage through the VESC, and determines whether a pit stop is required.

When the STOP sign is detected:

- If the battery is healthy, the vehicle continues autonomous driving.
- If the battery is low, the vehicle initiates an autonomous pit-stop sequence.
- The vehicle enters the pit area, stops for service, exits the pit, reacquires the lane, and resumes autonomous driving.

## System Architecture

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
