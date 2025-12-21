---
title: "Two Cops - TurtleBot4 Multi-Robot Navigation & Tracking System"
short_title: "Two Cops"
date: 2025-12-21
collection: project
permalink: /projects/two-cops-turtlebot4/
excerpt: "TurtleBot4 2대를 이용한 자율 네비게이션 및 YOLO 기반 객체 추적 시스템. Nav2 네비게이션과 딥러닝 비전을 결합한 다중 로봇 협업 프로젝트"
---

# 🚔 Two Cops - TurtleBot4 Multi-Robot System

> **프로젝트 이름**: Two Cops  
> **기간**: 2025  
> **플랫폼**: ROS2 Humble · TurtleBot4 · OAK-D Camera  
> **GitHub**: [Rokey-C1/turtlebot4_ws](https://github.com/Rokey-C1/turtlebot4_ws)

---

## 🎯 프로젝트 개요

**Two Cops**는 TurtleBot4 로봇 2대를 이용한 자율 네비게이션 및 실시간 객체 추적 시스템입니다. "경찰 2명"처럼 협력하는 다중 로봇 시스템으로, SLAM 기반 자율 주행과 YOLO 딥러닝을 활용한 객체 추적 기능을 통합했습니다.

### 핵심 기능
- 🗺️ **자율 네비게이션**: Nav2 기반 목표 지점 자동 이동
- 👁️ **실시간 객체 감지**: YOLO v8 + OAK-D 3D 카메라
- 🤖 **다중 로봇 제어**: 2대의 로봇 독립적 운용
- 📍 **정밀 위치 추정**: SLAM 맵 기반 로컬라이제이션

---

## 🏗️ 시스템 아키텍처

### 전체 구조
```
┌─────────────────────────────────────────────────────────┐
│                    ROS2 Network                         │
├─────────────────┬───────────────────┬───────────────────┤
│   Robot 1       │   Robot 2         │   Base Station    │
│  (/robot1)      │  (/robot2)        │   (Control PC)    │
├─────────────────┼───────────────────┼───────────────────┤
│ • Localization  │ • Localization    │ • RViz           │
│ • Navigation    │ • Navigation      │ • Launch Files   │
│ • YOLO Tracking │ • YOLO Tracking   │ • Map Server     │
│ • OAK-D Camera  │ • OAK-D Camera    │ • Monitoring     │
└─────────────────┴───────────────────┴───────────────────┘
```

### 패키지 구성
```
turtlebot4_ws/
├── src/
│   ├── turtlebot4              # TurtleBot4 공식 드라이버
│   ├── turtlebot4_simulator    # Gazebo 시뮬레이터
│   ├── turtlebot4_desktop      # RViz 시각화 툴
│   ├── m-explore-ros2          # 자율 탐색 (미지의 영역)
│   ├── turtlebot4_nav2pose     # 🔧 커스텀: 통합 네비게이션
│   ├── turtlebot4_tracking     # 🔧 커스텀: YOLO 객체 추적
│   └── twocops_bringup         # 🔧 커스텀: 통합 실행 런처
├── maps/                       # SLAM 생성 환경 지도
│   ├── key_map.yaml           # 메인 지도
│   ├── local2.yaml            # 로컬라이제이션 설정
│   └── nav2_net2.yaml         # Nav2 파라미터
└── model/
    └── best.pt                # YOLO 학습 모델
```

---

## 🚀 주요 기능

### 1. NavToPose - 자율 네비게이션

**목표**: 지정된 좌표로 로봇이 자동으로 이동하며 장애물을 회피합니다.

**기술 스택**:
- **Nav2**: ROS2 공식 네비게이션 스택
- **AMCL**: Adaptive Monte Carlo Localization (위치 추정)
- **DWB Controller**: Dynamic Window Approach (경로 추종)
- **Costmap**: 장애물 지도 생성

**실행 방법**:
```bash
# 1. 준비 상태 확인
./prep_checker.sh --n 1  # robot1

# 2. 통합 런치 (로컬라이제이션 + 네비게이션)
ros2 launch turtlebot4_nav2pose nav2pose.launch.py \
    namespace:=/robot1 \
    map:=$HOME/turtlebot4_ws/maps/key_map.yaml \
    params_file:=$HOME/turtlebot4_ws/maps/local2.yaml \
    nav2_params_file:=$HOME/turtlebot4_ws/maps/nav2_net2.yaml \
    initial_x:=0.09 initial_y:=0.67 initial_yaw:=1.57

# 3. 목표 위치 전송
ros2 action send_goal /robot1/navigate_to_pose \
    nav2_msgs/action/NavigateToPose \
    "pose: {header: {frame_id: map}, 
            pose: {position: {x: 2.3, y: 2.3, z: 0.0}, 
                   orientation: {w: 1.0}}}"
```

**주요 토픽**:
- `/robot<n>/cmd_vel` - 속도 명령
- `/robot<n>/amcl_pose` - 추정된 로봇 위치
- `/robot<n>/map` - SLAM 지도
- `/robot<n>/scan` - LiDAR 센서 데이터

---

### 2. Detection & Tracking - 객체 추적

**목표**: YOLO로 실시간 객체를 감지하고, 3D 위치를 계산하여 로봇이 자동으로 추적합니다.

**기술 스택**:
- **YOLOv8**: 딥러닝 객체 감지 (PyTorch)
- **OAK-D Camera**: RGB-D 스테레오 카메라
- **3D Position Estimation**: 카메라 좌표 → 로봇 좌표 변환

**핵심 알고리즘**:
```python
# 1. YOLO 객체 감지 (RGB 이미지)
results = model(rgb_image)
bbox = results[0].boxes  # Bounding box

# 2. Depth 값 추출 (중심점)
center_x = (bbox.x1 + bbox.x2) / 2
center_y = (bbox.y1 + bbox.y2) / 2
depth_value = depth_image[int(center_y), int(center_x)]

# 3. 3D 좌표 계산 (카메라 내부 파라미터 사용)
X = (center_x - cx) * depth / fx
Y = (center_y - cy) * depth / fy
Z = depth

# 4. 로봇 좌표계로 변환
distance = sqrt(X^2 + Y^2 + Z^2)
angle = atan2(X, Z)
```

**실행 방법**:
```bash
ros2 launch turtlebot4_tracking yolo_move_robot.launch.py \
    namespace:=robot1 \
    model_path:=/home/rokey/turtlebot4_ws/model/best.pt \
    rgb_topic:=/robot1/oakd/rgb/image_raw/compressed \
    depth_topic:=/robot1/oakd/stereo/image_raw \
    cam_info_topic:=/robot1/oakd/rgb/camera_info
```

**퍼블리시 토픽**:
- `/detected_x` (Float32) - 객체의 X 좌표 (카메라 기준)
- `/detected_angle` (Float32) - 객체 방향 (라디안)
- `/detected_distance` (Float32) - 객체까지의 거리 (미터)
- `/object_detected` (Bool) - 객체 감지 여부
- `/yolo_annotated_img` (Image) - YOLO 바운딩 박스가 그려진 이미지

**성능 최적화**:
- FPS 제한: 10fps (CPU 부하 감소)
- GPU 가속: CUDA 사용 (가능 시)
- 이미지 크기: 416x416 (YOLO 입력)
- Queue 기반 프레임 처리 (프레임 드롭 방지)

---

## 💻 코드 구조

### 1. turtlebot4_nav2pose (자율 네비게이션)

**nav2pose_node.py**:
```python
from turtlebot4_navigation.turtlebot4_navigator import TurtleBot4Navigator

def main():
    navigator = TurtleBot4Navigator()
    
    # 초기 위치 설정
    initial_pose = navigator.getPoseStamped([0.0, 0.0], TurtleBot4Directions.NORTH)
    navigator.setInitialPose(initial_pose)
    
    # Nav2 준비 대기
    navigator.waitUntilNav2Active()
    
    # 목표 위치 설정 및 이동
    goal_pose = navigator.getPoseStamped([-1.55, 0.066], TurtleBot4Directions.EAST)
    navigator.startToPose(goal_pose)
```

**launch/nav2pose.launch.py**:
- Localization 실행
- Navigation 실행
- 초기 위치 자동 설정
- 파라미터 통합 관리

---

### 2. turtlebot4_tracking (객체 추적)

**detection_node.py**:
```python
class DetectionNode(Node):
    def __init__(self):
        # YOLO 모델 로드
        self.model = YOLO(model_path)
        self.device = 'cuda' if torch.cuda.is_available() else 'cpu'
        
        # FPS 제한 (10fps)
        self.max_fps = 10.0
        self.last_process_time = 0.0
        
        # 구독자
        self.create_subscription(CompressedImage, rgb_topic, self.rgb_callback)
        self.create_subscription(Image, depth_topic, self.depth_callback)
        self.create_subscription(CameraInfo, cam_info_topic, self.camera_info_callback)
        
        # 퍼블리셔
        self.x_pub = self.create_publisher(Float32, 'detected_x', 10)
        self.angle_pub = self.create_publisher(Float32, 'detected_angle', 10)
        self.dist_pub = self.create_publisher(Float32, 'detected_distance', 10)
        self.detected_pub = self.create_publisher(Bool, 'object_detected', 10)
    
    def process_thread_fn(self):
        """YOLO 추론 스레드 (메인 루프와 분리)"""
        while rclpy.ok():
            # Queue에서 프레임 가져오기
            rgb_image = self.rgb_queue.get()
            
            # FPS 제한 체크
            current_time = time.time()
            if (current_time - self.last_process_time) < (1.0 / self.max_fps):
                continue
            
            # YOLO 추론
            results = self.model(rgb_image)
            
            # 3D 위치 계산 및 퍼블리시
            self.calculate_3d_position(results)
```

**tracking_node.py**:
```python
class TrackingNode(Node):
    """감지된 객체를 따라가는 제어 노드"""
    
    def __init__(self):
        self.create_subscription(Float32, 'detected_angle', self.angle_callback)
        self.create_subscription(Float32, 'detected_distance', self.distance_callback)
        self.cmd_vel_pub = self.create_publisher(Twist, 'cmd_vel', 10)
    
    def angle_callback(self, msg):
        """각도에 따라 회전 속도 조정"""
        twist = Twist()
        twist.angular.z = -msg.data * 0.5  # P 제어
        self.cmd_vel_pub.publish(twist)
    
    def distance_callback(self, msg):
        """거리에 따라 전진 속도 조정"""
        twist = Twist()
        if msg.data > 0.5:  # 0.5m 이상이면 전진
            twist.linear.x = min((msg.data - 0.5) * 0.3, 0.2)
        self.cmd_vel_pub.publish(twist)
```

---

## 🗺️ 맵 생성 (SLAM)

**사전 작업**: 환경 지도를 SLAM으로 생성

```bash
# 1. SLAM 툴박스 실행
ros2 launch turtlebot4_navigation slam.launch.py namespace:=/robot1

# 2. 로봇을 수동 조작하여 맵 생성
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r /cmd_vel:=/robot1/cmd_vel

# 3. 맵 저장
ros2 run nav2_map_server map_saver_cli -f ~/turtlebot4_ws/maps/key_map
```

**생성 파일**:
- `key_map.yaml` - 맵 메타데이터
- `key_map.pgm` - 맵 이미지 (점유 그리드)

---

## 🛠️ 시스템 요구사항

### 하드웨어
- **TurtleBot4 Standard** x2
  - Create 3 Mobile Base
  - OAK-D Camera (RGB-D)
  - RPLidar A1 (2D LiDAR)
  - Raspberry Pi 4B (4GB RAM)

- **Base Station**
  - CPU: Intel i5 이상
  - GPU: NVIDIA GTX 1060 이상 (YOLO용)
  - RAM: 16GB 이상
  - Ubuntu 22.04 LTS

### 소프트웨어
- ROS2 Humble Hawksbill
- Nav2 (navigation2)
- YOLOv8 (ultralytics)
- PyTorch 2.0+
- OpenCV 4.x
- Python 3.10+

---

## 📊 성능 지표

| 항목 | 측정값 |
|------|--------|
| YOLO 추론 속도 | ~10 FPS (GPU) |
| 네비게이션 정확도 | ±5cm (AMCL) |
| 객체 감지 거리 | 최대 3m (OAK-D) |
| 맵 업데이트 주기 | 10Hz (Costmap) |
| 동시 로봇 제어 | 2대 (확장 가능) |

---

## 🎓 배운 점 & 성과

### 기술적 성과
1. **다중 로봇 네임스페이스 관리**
   - ROS2의 namespace 기능을 활용한 깔끔한 토픽 구조
   - `/robot1`, `/robot2`로 완전히 분리된 제어

2. **YOLO + ROS2 통합**
   - 딥러닝 추론과 ROS2 노드를 멀티스레딩으로 통합
   - Queue 기반 프레임 처리로 안정성 확보

3. **Nav2 파라미터 튜닝**
   - Costmap inflation radius 조정
   - DWB 속도 프로파일 최적화
   - Recovery behaviors 설정

### 개선 가능한 점
- [ ] 다중 로봇 간 협업 알고리즘 (Task allocation)
- [ ] YOLO 모델 경량화 (TensorRT 최적화)
- [ ] 실시간 SLAM (Cartographer 전환)
- [ ] 웹 기반 모니터링 대시보드

---

## 🔗 관련 링크

- **GitHub Repository**: [Rokey-C1/turtlebot4_ws](https://github.com/Rokey-C1/turtlebot4_ws)
- **TurtleBot4 공식 문서**: [turtlebot.github.io](https://turtlebot.github.io/turtlebot4-user-manual/)
- **Nav2 문서**: [navigation.ros.org](https://navigation.ros.org/)
- **YOLOv8**: [ultralytics.com](https://docs.ultralytics.com/)

---

## 📝 라이선스

This project includes components from:
- TurtleBot4 (Apache 2.0)
- Nav2 (Apache 2.0)
- YOLOv8 (AGPL-3.0)

Custom packages (turtlebot4_nav2pose, turtlebot4_tracking) are licensed under Apache 2.0.

---

**Last Updated**: 2025-12-21  
**Contact**: [GitHub @Rokey-C1](https://github.com/Rokey-C1)
