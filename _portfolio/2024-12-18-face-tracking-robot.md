---
title: "Face Tracking Robot — Web Control System"
short_title: "Face-Tracking Project"
date: 2024-12-18
collection: portfolio
type: Project
permalink: /projects/face-tracking-robot/
teaser: homepage.png
excerpt: "ROS2 기반 얼굴 추적 로봇의 전체 아키텍처: ROS2 노드, Web UI, IPC, 비전·음성 파이프라인을 통합한 실전형 설계 문서."
read_time: true
share: true
categories: [Project, Robotics]
tags: [ROS2, Face Tracking, Web, Doosan, RealSense]
---

# 🤖 Face Tracking Robot Web Control System Architecture

> **Version**: 2.0  
> **Date**: 2024-12-18  
> **System**: ROS2 Humble · Doosan M0609 · OnRobot RG2 · RealSense D435i

---

## 🧭 Overview

이 문서는 **얼굴 추적 기반 로봇 웹 제어 시스템**의 전체 아키텍처를 정리한 공식 설계 문서입니다.  
ROS2 기반 로봇 제어, 웹 UI, 파일 기반 IPC, 비전·음성·시나리오 파이프라인까지 **실제 시연 가능한 구조**를 기준으로 작성되었습니다.

---

## 📦 Required Packages

### ✅ Core Packages (필수)

| Package                        | Path                     | Role                                      |
| ------------------------------ | ------------------------ | ----------------------------------------- |
| **bbangee**                    | `src/bbangee/`           | Web App (FastAPI + React)                 |
| **doosan-robot2**              | `src/doosan-robot2/`     | Doosan ROS2 driver (ros2_control)         |
| **gripper_rviz_sync**          | `src/gripper_rviz_sync/` | RG2 Modbus control + RViz sync            |
| **gripper_camera_description** | `rokey_packages/`        | Unified URDF (Robot + Gripper + Camera)   |
| **camera_utils**               | `rokey_packages/`        | Camera preprocessing (flip)               |
| **face_tracking**              | `rokey_packages/`        | Face detection / tracking / joint control |
| **ros2_web_bridge**            | `rokey_packages/`        | ROS2 ↔ Web bridge                         |
| **obb**                        | `src/obb/`               | YOLO OBB model (armband detection)        |

### ⚙️ Optional Packages

| Package             | Path              | Role                 |
| ------------------- | ----------------- | -------------------- |
| **voice_auth**      | `rokey_packages/` | Voice authentication |
| **voice_auth_msgs** | `rokey_packages/` | Voice auth messages  |

### 🔗 External Dependencies

| Package               | Install | Role              |
| --------------------- | ------- | ----------------- |
| realsense2_camera     | apt     | RealSense driver  |
| controller_manager    | apt     | ros2_control      |
| robot_state_publisher | apt     | TF / URDF publish |

### ❌ Archived / Unused

| Package             | Reason                      |
| ------------------- | --------------------------- |
| archive             | legacy backup               |
| band_cnn            | replaced by OBB + OCR       |
| bbox                | experiment                  |
| collision_recovery  | merged into ros2_web_bridge |
| gripper_control     | deprecated                  |
| sam3_grip_detection | not in current scenario     |

---

## 📁 Project Structure

```text
ros2_ws/src/
├── bbangee/                    # 🌐 Web
├── rokey_packages/             # 📦 Custom ROS2
│   ├── camera_utils/
│   ├── face_tracking/
│   ├── gripper_camera_description/
│   ├── ros2_web_bridge/
│   ├── voice_auth/
│   └── voice_auth_msgs/
├── gripper_rviz_sync/
├── doosan-robot2/
├── external/
└── archive/
```

---

## 🚀 Startup Scripts

### ▶️ Full System

```bash
./bbangee/start_all.sh
```

### ▶️ Hardware Only

```bash
./bbangee/start_hardware.sh [ROBOT_IP] [MODE] [GRIPPER_IP]
```

### ▶️ ROS2 Nodes Only

```bash
./bbangee/start_ros2_nodes.sh
```

### ▶️ Web Only

```bash
./bbangee/start_web.sh
```

---

## 🧠 Core Architecture

### 1️⃣ Unified URDF

**Package**: gripper_camera_description  
**Launch**: `dsr_bringup2_with_gripper.launch.py`

**Nodes**

* ros2_control_node
* robot_state_publisher
* joint_trajectory_controller
* run_emulator (virtual)

---

### 2️⃣ Gripper Control (RG2)

**Package**: gripper_rviz_sync

**Topics**

* `/gripper/command`
* `/gripper/width/current`
* `/gripper/grip_detected`

**Modbus**: 192.168.1.1:502

---

### 3️⃣ Camera Pipeline

```text
RealSense → image_flip → face_detection → face_tracking → joint_tracking
```

---

### 4️⃣ Face Tracking

| Node                | Role                       |
| ------------------- | -------------------------- |
| face_detection_node | MediaPipe detection        |
| face_tracking_node  | target selection           |
| joint_tracking_node | IK + FollowJointTrajectory |

---

### 5️⃣ ROS2 ↔ Web Bridge

| Node               | Role                  |
| ------------------ | --------------------- |
| bridge_node        | state → JSON          |
| robot_controller   | command.json → Action |
| camera_streamer    | WebSocket video       |
| collision_recovery | collision handling    |

---

## 🌐 Web Application

### Backend (FastAPI :8000)

| Router      | Purpose       |
| ----------- | ------------- |
| ros2.py     | state         |
| robot.py    | motion        |
| gripper.py  | gripper       |
| devices.py  | ESP32         |
| armband.py  | OBB + OCR     |
| scenario.py | state machine |
| voice.py    | STT/TTS       |

### Frontend (React :5173)

* RobotPanel
* GripperControl
* CameraPanel
* CollisionPanel
* ServoControl
* ScenarioPanel

---

## 🔌 Network Map

| Device       | IP             | Protocol |
| ------------ | -------------- | -------- |
| Doosan M0609 | 192.168.1.100  | TCP      |
| OnRobot RG2  | 192.168.1.1    | Modbus   |
| ESP32        | 192.168.10.50  | HTTP     |
| Backend      | localhost:8000 | HTTP     |
| Frontend     | localhost:5173 | HTTP     |

---

## 🔁 IPC Files

```json
/tmp/ros2_bridge_state.json
/tmp/ros2_bridge_command.json
/tmp/gripper_state.json
```

---

## 🎬 Scenario State Machine

```text
IDLE → DETECTED → PASSWORD_CHECK →
	├─ ALLY_PASS
	├─ ALLY_ALERT
	├─ ENEMY_CRITICAL
	└─ ENEMY_ENGAGE
```

---

## 🎤 Voice Authentication

* TTS: ElevenLabs
* STT: Google Speech
* Challenge: 로키
* Response: 협동

---

## 🔍 Armband OCR Pipeline

YOLO OBB → Perspective Transform → EasyOCR → 3x Consensus → Scenario

---

## 📊 Logging

| Component | Path                   |
| --------- | ---------------------- |
| Robot     | /tmp/ros2_bringup.log  |
| Gripper   | /tmp/ros2_gripper.log  |
| Camera    | /tmp/ros2_camera.log   |
| Tracking  | /tmp/ros2_tracking.log |
| Web       | /tmp/backend.log       |

---

## 🧪 Testing

```bash
ros2 topic echo /dsr01/joint_states
ros2 topic echo /gripper/width/current
ros2 topic hz /camera/image_flipped
```

---

## 📝 Changelog

* **v2.0**: unified gripper command, UI simplification
* **v1.0**: initial system

---

## 📎 References

* CONTROL_METHODS_COMPARISON.md
* PERFORMANCE_OPTIMIZATION.md
* REALSENSE_FILTER_COMPARISON.md

---

이 페이지는 포트폴리오 항목으로 `/projects/face-tracking-robot/`에 추가되었습니다.
---

## 핵심 기능

- 통합 URDF 및 `ros2_control` 기반 로봇 제어
- RG2 그리퍼 Modbus 제어 + RViz 동기화
- RealSense → MediaPipe → Face Tracking → Joint Tracking 파이프라인
- ROS2 ↔ Web 브리지(상태 JSON, 명령 파일 기반 IPC, WebSocket 비디오)
- FastAPI 백엔드 + React 프런트엔드의 실시간 제어 인터페이스

---

## 아키텍처 요약

프로젝트 구조 및 주요 노드, 네트워크 맵, IPC 및 시나리오 상태 머신을 포함하여 실전 운영을 목표로 설계했습니다. 하드웨어/소프트웨어 패키지 목록, 네트워크 구성, 로그/테스트 명령 등도 문서화되어 있어 배포·시연에 바로 활용 가능합니다.

더 자세한 문서는 저장소 내 README 및 개별 패키지 문서를 참고하세요.
