# Machine Exercise 1: Red Light, Green Light

## Overview
This project implements a computer vision–based version of the classic **Red Light, Green Light** game using **Python**, **OpenCV**, and **YOLOv8**.  
Players are monitored through a webcam, and movement is detected in real-time. When the signal is **green**, players can move forward; when the signal turns **red**, any detected motion results in elimination.

---

## Features
- Real-time motion detection using **OpenCV**.
- Object/person detection powered by **YOLOv8**.
- State machine logic for switching between **Red Light** and **Green Light** phases.
- Visual feedback on screen (text and bounding boxes).
- Extendable framework for adding custom rules or integrating with other CV models.
