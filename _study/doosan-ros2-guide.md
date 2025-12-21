---
title: "두산 로봇 ROS2 패키지 가이드"
collection: study
type: "Guide"
permalink: /study/doosan-ros2-guide
date: 2025-12-21
---

# 두산 로보틱스 ROS2 완벽 가이드

> 최신 공식 패키지 기준 (doosan-robot2, ROS2 Humble)  
> 작성일: 2025년 12월 21일

---

## 📋 목차

1. [개요](#1-개요)
2. [시스템 요구사항](#2-시스템-요구사항)
3. [설치 방법](#3-설치-방법)
4. [실제 로봇 하드웨어 준비](#4-실제-로봇-하드웨어-준비)
5. [로봇 연결 및 실행](#5-로봇-연결-및-실행)
6. [패키지 구조 상세](#6-패키지-구조-상세)
7. [주요 기능 및 API](#7-주요-기능-및-api)
8. [예제 코드 분석](#8-예제-코드-분석)
9. [트러블슈팅](#9-트러블슈팅)

---

## 1. 개요

### 1.1 두산 로보틱스 ROS2 패키지란?

두산 로보틱스의 모든 협동로봇 모델을 ROS2 Humble 환경에서 제어할 수 있는 공식 패키지입니다.

**지원 로봇 모델:**
- M 시리즈: M0609, M0617, M1013, M1509
- H 시리즈: H2017, H2515
- A 시리즈: A0509s, A0912s
- E 시리즈: E0509, E0509s

### 1.2 주요 특징

- ✅ **Real/Virtual 모드** - 실제 로봇 또는 가상 시뮬레이터 제어
- ✅ **Docker 기반 에뮬레이터** - 별도 하드웨어 없이 테스트 가능
- ✅ **Gazebo 시뮬레이션** - 물리 시뮬레이션 환경
- ✅ **MoveIt2 통합** - 모션 플래닝 및 경로 계획
- ✅ **RViz2 시각화** - 실시간 로봇 상태 모니터링

---

## 2. 시스템 요구사항

### 2.1 필수 환경

```bash
OS: Ubuntu 22.04 LTS
ROS2: Humble Hawksbill
Python: 3.10+
Docker: (Virtual 모드 사용 시)
```

### 2.2 하드웨어 요구사항

**실제 로봇 연결 시:**
- 두산 로봇 컨트롤러와 네트워크 연결
- 기본 IP: `192.168.137.100`
- 기본 포트: `12345`

---

## 3. 설치 방법

### 3.1 의존성 설치

```bash
# 시스템 업데이트
sudo apt-get update

# 필수 패키지 설치
sudo apt-get install -y libpoco-dev libyaml-cpp-dev wget \
                        ros-humble-control-msgs ros-humble-realtime-tools ros-humble-xacro \
                        ros-humble-joint-state-publisher-gui ros-humble-ros2-control \
                        ros-humble-ros2-controllers ros-humble-gazebo-msgs ros-humble-moveit-msgs \
                        dbus-x11 ros-humble-moveit-configs-utils ros-humble-moveit-ros-move-group \
                        ros-humble-gazebo-ros-pkgs ros-humble-ros-gz-sim ros-humble-ign-ros2-control
```

### 3.2 Gazebo 시뮬레이션 설치

```bash
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y libignition-gazebo6-dev ros-humble-gazebo-ros-pkgs ros-humble-ros-gz-sim ros-humble-ros-gz
```

### 3.3 Docker 설치 (Virtual 모드용)

```bash
# Docker 공식 가이드 참고
# https://docs.docker.com/engine/install/ubuntu/

# 빠른 설치
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

### 3.4 패키지 다운로드 및 빌드

```bash
# 워크스페이스 생성
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 두산 로보틱스 패키지 클론
git clone -b humble https://github.com/doosan-robotics/doosan-robot2.git

# 의존성 설치
cd ~/ros2_ws
rosdep install -r --from-paths . --ignore-src --rosdistro $ROS_DISTRO -y

# 에뮬레이터 설치
cd ~/ros2_ws/src/doosan-robot2
chmod +x ./install_emulator.sh
sudo ./install_emulator.sh

# 패키지 빌드
cd ~/ros2_ws
colcon build
source install/setup.bash
```

### 3.5 환경 설정

**~/.bashrc에 추가:**
```bash
# ROS2 환경
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

# DSR 모듈 경로 (Python 예제 사용 시)
export PYTHONPATH=$PYTHONPATH:~/ros2_ws/install/dsr_common2/lib/dsr_common2/imp
```
적용:
```bash
source ~/.bashrc
```

---

## 4. 실제 로봇 하드웨어 준비

> ⚠️ **이 섹션은 실제 두산 로봇을 사용할 때만 필요합니다.**  
> Virtual Mode로 시뮬레이션만 할 경우 [5. 로봇 연결 및 실행](#5-로봇-연결-및-실행)으로 바로 이동하세요.

### 4.1 로봇 전원 켜기

#### 1단계: 전원 연결 확인
```
1. 로봇 베이스 뒷면의 전원 케이블 연결 확인
2. 컨트롤 박스(컨트롤러) 전원 케이블 연결 확인
3. Emergency Stop 버튼이 눌려있지 않은지 확인
```

#### 2단계: 컨트롤러 전원 켜기
```
1. 컨트롤 박스의 전원 스위치를 ON으로 설정
2. 부팅 소리가 들리고 LED가 켜짐 (약 30초 소요)
3. 티치펜던트(Teach Pendant) 화면이 켜질 때까지 대기
```

#### 3단계: 로봇 초기화
```
1. 티치펜던트에서 "Initialize Robot" 또는 초기화 메뉴 선택
2. 로봇이 자동으로 영점(Zero Position) 탐색
3. 초기화 완료까지 약 1-2분 소요
```

### 4.2 티치펜던트 기본 조작

#### 메인 화면 구성
```
┌─────────────────────────────┐
│  DOOSAN ROBOTICS            │
│                             │
│  [Manual]  [Auto]  [Stop]   │
│                             │
│  Robot Status: Ready        │
│  Mode: Manual               │
│  Safety: Normal             │
│                             │
│  [Settings] [Network] [I/O] │
└─────────────────────────────┘
```

#### 주요 버튼 설명
- **Manual Mode**: 수동 조작 모드 (조그 이동 가능)
- **Auto Mode**: 자동 실행 모드 (프로그램 실행)
- **Emergency Stop**: 빨간색 버튼 - 즉시 정지
- **Enable Switch**: 티치펜던트 측면 - 로봇 활성화

### 4.3 네트워크 설정 (중요!)

#### 방법 1: 티치펜던트에서 설정

```
1. 티치펜던트에서 [Settings] → [Network] 선택
2. IP Address 설정 확인/변경
   - 기본값: 192.168.137.100
   - Subnet: 255.255.255.0
   - Gateway: 192.168.137.1
3. [Apply] 버튼으로 저장
4. 로봇 재부팅 (설정 적용을 위해)
```

#### 방법 2: PC 네트워크 설정

**Ubuntu에서 PC IP 설정:**

```bash
# 1. 네트워크 설정 열기
nm-connection-editor

# 또는 GUI: Settings → Network → Wired → Settings (톱니바퀴)

# 2. IPv4 탭 선택
#    - Method: Manual
#    - Address: 192.168.137.10 (또는 .2 ~ .254 중 선택)
#    - Netmask: 255.255.255.0
#    - Gateway: 192.168.137.1

# 3. 저장 후 네트워크 재연결
```

**연결 확인:**
```bash
# 로봇 Ping 테스트
ping 192.168.137.100

# 성공 시 출력:
# 64 bytes from 192.168.137.100: icmp_seq=1 ttl=64 time=0.5 ms
```

### 4.4 로봇 컨트롤러 모드 설정

#### Autonomous Mode 활성화 (ROS2 제어용)

```
1. 티치펜던트에서 [Settings] → [System] 선택
2. [Control Mode] → [External Control] 설정
3. [Protocol] → TCP/IP 선택
4. [Port] → 12345 확인
5. [Autonomous Mode] 활성화
6. [Apply] 저장
```

⚠️ **중요:** ROS2로 제어하려면 반드시 **Autonomous Mode**가 활성화되어 있어야 합니다!

### 4.5 안전 설정 확인

#### Safety Zone 설정
```
1. [Settings] → [Safety] → [Safety Zone]
2. 작업 범위 확인 (필요 시 조정)
3. Collision Detection 활성화 권장
```

#### Speed Limit 설정
```
1. [Settings] → [Safety] → [Speed Limit]
2. 초기 테스트 시 속도 제한 권장 (예: 50%)
3. 익숙해지면 점차 증가
```

### 4.6 케이블 연결

#### Ethernet 케이블 연결

```
PC Ethernet Port  ←→  Robot Controller Ethernet Port
                      (보통 "EXT" 또는 "EXTERNAL" 표시)
```

**케이블 연결 위치:**
- 컨트롤 박스 뒷면 또는 측면
- "EXTERNAL CONTROL" 또는 "PC" 라벨 확인
- RJ45 Ethernet 포트 사용

### 4.7 첫 연결 테스트

#### 1단계: 네트워크 연결 확인
```bash
# 로봇 IP로 Ping
ping 192.168.137.100

# 포트 확인
nc -zv 192.168.137.100 12345
# 또는
telnet 192.168.137.100 12345
```

#### 2단계: ROS2로 로봇 상태 확인
```bash
# 터미널 1: 로봇 연결
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=real \
    host:=192.168.137.100 \
    port:=12345 \
    model:=m1013

# 터미널 2: Joint State 확인
ros2 topic echo /dsr01/joint_states
```

**정상 연결 시 출력 예시:**
```yaml
header:
  stamp:
    sec: 1234567890
    nanosec: 123456789
  frame_id: ''
name:
- joint1
- joint2
- joint3
- joint4
- joint5
- joint6
position: [0.0, 0.0, 1.57, 0.0, 1.57, 0.0]
velocity: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
effort: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
```

### 4.8 초기 세팅 체크리스트

로봇을 처음 사용하기 전 확인사항:

- [ ] ✅ 로봇 전원 켜짐 및 초기화 완료
- [ ] ✅ Emergency Stop 해제됨
- [ ] ✅ 티치펜던트 정상 작동
- [ ] ✅ Autonomous Mode 활성화
- [ ] ✅ 네트워크 설정 완료 (IP: 192.168.137.100)
- [ ] ✅ PC와 로봇 Ping 통신 성공
- [ ] ✅ Port 12345 열려있음
- [ ] ✅ 작업 공간 안전 확인
- [ ] ✅ 충돌 가능성 있는 장애물 제거
- [ ] ✅ Safety Zone 설정 확인

### 4.9 자주 하는 실수

#### ❌ Emergency Stop 활성화 상태로 연결 시도
**증상:** "Robot is in emergency stop state" 에러  
**해결:** E-Stop 버튼을 시계방향으로 돌려서 해제

#### ❌ Autonomous Mode 비활성화
**증상:** "Robot is not in autonomous mode" 에러  
**해결:** 티치펜던트에서 Autonomous Mode 활성화

#### ❌ 잘못된 IP 주소
**증상:** "Connection refused" 또는 "Timeout"  
**해결:** 
- 티치펜던트에서 IP 확인
- PC IP가 같은 서브넷인지 확인 (192.168.137.x)

#### ❌ 방화벽 차단
**증상:** Ping은 되는데 ROS2 연결 안됨  
**해결:**
```bash
sudo ufw allow 12345/tcp
sudo ufw reload
```

---

## 5. 로봇 연결 및 실행

### 5.1 실행 모드

두산 로봇은 **2가지 모드**로 실행 가능합니다:

#### 🔴 Real Mode (실제 로봇)
- 실제 두산 로봇 컨트롤러와 네트워크 연결
- 로봇 IP 주소 필요 (기본: 192.168.137.100)

#### 🟢 Virtual Mode (시뮬레이터)
- Docker 기반 에뮬레이터 자동 실행
- 로봇 없이 테스트 가능
- IP: 127.0.0.1

### 5.2 기본 실행 방법

#### ✅ Virtual Mode + RViz2 (가장 기본)

```bash
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12345 \
    model:=m1013
```

**설명:**
- `mode:=virtual` - 가상 시뮬레이터 사용
- `host:=127.0.0.1` - 로컬 에뮬레이터 연결
- `port:=12345` - 통신 포트
- `model:=m1013` - 로봇 모델 (m0609, m1013, m1509 등)

#### ✅ Real Mode + RViz2 (실제 로봇)

```bash
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=real \
    host:=192.168.137.100 \
    port:=12345 \
    model:=m1013
```

### 5.3 Gazebo 시뮬레이션 실행

#### Virtual Mode + Gazebo

```bash
ros2 launch dsr_bringup2 dsr_bringup2_gazebo.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12346 \
    model:=m1013 \
    name:=dsr01 \
    x:=0 \
    y:=0
```

**다중 로봇 추가:**
```bash
# 두 번째 로봇 추가
ros2 launch dsr_bringup2 dsr_bringup2_spawn_on_gazebo.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12347 \
    name:=dsr02 \
    x:=2 \
    y:=2
```

⚠️ **주의:** 각 로봇은 고유한 `port`, `name`, `x`, `y` 값이 필요합니다.

### 5.4 MoveIt2 실행

**필수 조건:** 컨트롤러 버전 2.12 이상

```bash
# Virtual Mode
ros2 launch dsr_bringup2 dsr_bringup2_moveit.launch.py \
    mode:=virtual \
    model:=m1013 \
    host:=127.0.0.1

# Real Mode
ros2 launch dsr_bringup2 dsr_bringup2_moveit.launch.py \
    mode:=real \
    model:=m1013 \
    host:=192.168.137.100
```

### 5.5 주요 Launch 파라미터

| 파라미터 | 설명 | 기본값 | 예시 |
|---------|------|--------|------|
| `mode` | real / virtual | real | `mode:=virtual` |
| `host` | 로봇 IP 주소 | 192.168.137.100 | `host:=127.0.0.1` |
| `port` | 통신 포트 | 12345 | `port:=12345` |
| `model` | 로봇 모델명 | - | `model:=m1013` |
| `name` | 로봇 네임스페이스 | dsr01 | `name:=dsr01` |
| `color` | 로봇 색상 | white | `color:=blue` |
| `gui` | GUI 활성화 | true | `gui:=false` |
| `gz` | Gazebo 실행 | false | `gz:=true` |
| `x`, `y` | Gazebo 위치 | 0, 0 | `x:=2 y:=3` |

---

## 6. 패키지 구조 상세

### 6.1 전체 패키지 구조

```
doosan-robot2/
├── dsr_bringup2/          # Launch 파일 모음
├── dsr_common2/           # 핵심 라이브러리
│   └── imp/               # Python DSR 모듈
│       ├── DSR_ROBOT2.py  # 메인 로봇 제어 API
│       ├── DR_common2.py  # 공통 함수
│       ├── DR_init.py     # 초기화
│       └── ...
├── dsr_controller2/       # ROS2 컨트롤러
├── dsr_description2/      # URDF/XACRO 모델
├── dsr_example2/          # 예제 코드
│   ├── dsr_example/       # 기본 예제
│   ├── dsr_realtime_control/  # 실시간 제어
│   └── dsr_visualservoing/    # 비전 서보잉
├── dsr_gazebo2/           # Gazebo 시뮬레이션
├── dsr_hardware2/         # 하드웨어 인터페이스
├── dsr_moveit2/           # MoveIt2 설정
├── dsr_msgs2/             # 커스텀 메시지
├── dsr_mujoco/            # MuJoCo 시뮬레이션
└── dsr_tests/             # 테스트 코드
```

### 6.2 핵심 패키지 설명

#### 📦 dsr_bringup2
로봇 실행을 위한 Launch 파일들

**주요 Launch 파일:**
- `dsr_bringup2_rviz.launch.py` - RViz2 시각화
- `dsr_bringup2_gazebo.launch.py` - Gazebo 시뮬레이션
- `dsr_bringup2_moveit.launch.py` - MoveIt2 모션 플래닝
- `dsr_bringup2_spawn_on_gazebo.launch.py` - Gazebo에 추가 로봇 생성

#### 📦 dsr_common2
Python 제어 API가 포함된 핵심 라이브러리

**imp/ 디렉토리 주요 파일:**
- `DSR_ROBOT2.py` - 로봇 제어 함수 (movej, movel 등)
- `DR_common2.py` - 위치 정의 (posj, posx)
- `DR_init.py` - 로봇 초기화
- `DRFC.py` - Force Control
- `DR_error2.py` - 에러 처리

#### 📦 dsr_example2
사용자를 위한 예제 코드

**dsr_example/ 하위:**
- `simple/single_robot_simple.py` - 기본 단일 로봇 제어
- `demo/dance_m1013.py` - 다양한 동작 시연
- `demo/slope_demo.py` - 경사면 제어 예제

#### 📦 dsr_description2
로봇 URDF 모델 및 메시 파일

#### 📦 dsr_msgs2
두산 로봇 전용 ROS2 메시지 타입

---

## 7. 주요 기능 및 API

### 7.1 Python API 사용 준비

```python
import rclpy
import DR_init

# 로봇 설정
ROBOT_ID = "dsr01"
ROBOT_MODEL = "m1013"

DR_init.__dsr__id = ROBOT_ID
DR_init.__dsr__model = ROBOT_MODEL

# ROS2 노드 생성
rclpy.init()
node = rclpy.create_node('my_robot_node', namespace=ROBOT_ID)
DR_init.__dsr__node = node

# DSR 라이브러리 import
from DSR_ROBOT2 import (
    movej, movel, movec,
    set_robot_mode,
    set_velx, set_accx
)
from DR_common2 import posj, posx
```

### 7.2 위치 정의

#### Joint Position (관절 각도)
```python
from DR_common2 import posj

# 6축 관절 각도 (degree)
home_pos = posj(0, 0, 90, 0, 90, 0)
custom_pos = posj(10, -20, 45, 0, 60, 30)
```

#### Task Position (작업 공간 좌표)
```python
from DR_common2 import posx

# x, y, z (mm), rx, ry, rz (degree)
target = posx(400, 500, 600, 0, 180, 0)
```

#### Blend Position (블렌딩 동작)
```python
from DR_common2 import posb
from DSR_ROBOT2 import DR_LINE, DR_CIRCLE

# 직선 세그먼트
seg1 = posb(DR_LINE, posx(400, 500, 600, 0, 180, 0), radius=20)

# 원호 세그먼트 (via point 필요)
seg2 = posb(DR_CIRCLE, 
            posx(450, 550, 600, 0, 180, 0),  # via point
            posx(500, 600, 600, 0, 180, 0),  # end point
            radius=25)
```

### 7.3 모션 제어 함수

#### movej - 관절 공간 이동
```python
from DSR_ROBOT2 import movej

# 기본 사용
movej(posj(0, 0, 90, 0, 90, 0), vel=100, acc=100)

# 파라미터:
# - pos: 목표 관절 각도 (posj)
# - vel: 속도 (degree/sec)
# - acc: 가속도 (degree/sec²)
```

#### movel - 직선 이동
```python
from DSR_ROBOT2 import movel

# 직선 경로로 이동
movel(posx(400, 500, 600, 0, 180, 0), vel=[100, 50], acc=[200, 100])

# 파라미터:
# - pos: 목표 작업 좌표 (posx)
# - vel: [선속도(mm/sec), 회전속도(deg/sec)]
# - acc: [선가속도(mm/sec²), 회전가속도(deg/sec²)]
```

#### movec - 원호 이동
```python
from DSR_ROBOT2 import movec

# via point를 지나는 원호 이동
via = posx(450, 550, 600, 0, 180, 0)
target = posx(500, 600, 600, 0, 180, 0)
movec(via, target, vel=[100, 50], acc=[200, 100])
```

#### movejx - 작업 좌표로 관절 이동
```python
from DSR_ROBOT2 import movejx

# 작업 좌표를 지정하지만 관절 공간으로 이동
movejx(posx(400, 500, 600, 0, 180, 0), vel=60, acc=100, sol=0)

# sol: 역기구학 솔루션 번호 (0~7)
```

#### movesj - 다중 관절 경유점 이동
```python
from DSR_ROBOT2 import movesj

qlist = [
    posj(0, 0, 90, 0, 90, 0),
    posj(10, -10, 80, 0, 90, 0),
    posj(20, -20, 70, 0, 90, 0)
]
movesj(qlist, vel=100, acc=100)
```

#### movesx - 다중 작업 경유점 이동
```python
from DSR_ROBOT2 import movesx

xlist = [
    posx(400, 500, 600, 0, 180, 0),
    posx(450, 550, 600, 0, 180, 0),
    posx(500, 600, 600, 0, 180, 0)
]
movesx(xlist, vel=100, acc=100)
```

#### moveb - 블렌딩 이동
```python
from DSR_ROBOT2 import moveb, DR_BASE, DR_MV_MOD_ABS

b_list = [seg1, seg2, seg3]  # posb로 정의된 세그먼트 리스트
moveb(b_list, vel=150, acc=250, ref=DR_BASE, mod=DR_MV_MOD_ABS)
```

#### move_spiral - 나선 이동
```python
from DSR_ROBOT2 import move_spiral, DR_AXIS_Z, DR_TOOL

move_spiral(
    rev=9.5,      # 회전수
    rmax=20.0,    # 최대 반지름 (mm)
    lmax=50.0,    # 최대 길이 (mm)
    time=20.0,    # 시간 (sec)
    axis=DR_AXIS_Z,
    ref=DR_TOOL
)
```

#### move_periodic - 주기 운동
```python
from DSR_ROBOT2 import move_periodic, DR_TOOL

move_periodic(
    amp=[10, 0, 0, 0, 30, 0],  # 진폭 [x,y,z,rx,ry,rz]
    period=1.0,                 # 주기 (sec)
    atime=0.2,                  # 가속 시간 (sec)
    repeat=5,                   # 반복 횟수
    ref=DR_TOOL
)
```

### 7.4 로봇 상태 및 설정

#### 로봇 모드 설정
```python
from DSR_ROBOT2 import set_robot_mode, ROBOT_MODE_AUTONOMOUS

set_robot_mode(ROBOT_MODE_AUTONOMOUS)
```

#### 속도/가속도 설정
```python
from DSR_ROBOT2 import set_velx, set_accx

# 전역 작업 속도 설정
set_velx(30, 20)   # 선속도 30mm/sec, 회전속도 20deg/sec
set_accx(60, 40)   # 선가속도 60mm/sec², 회전가속도 40deg/sec²
```

#### TCP 및 Tool 설정
```python
from DSR_ROBOT2 import set_tcp, set_tool

set_tool("Tool Weight_2FG")
set_tcp("2FG_TCP")
```

### 7.5 상수 정의

```python
from DSR_ROBOT2 import (
    # 경로 타입
    DR_LINE,        # 직선
    DR_CIRCLE,      # 원호
    
    # 좌표계
    DR_BASE,        # 베이스 좌표계
    DR_TOOL,        # 툴 좌표계
    DR_WORLD,       # 월드 좌표계
    
    # 축
    DR_AXIS_X,
    DR_AXIS_Y,
    DR_AXIS_Z,
    
    # 이동 모드
    DR_MV_MOD_ABS,  # 절대 좌표
    DR_MV_MOD_REL,  # 상대 좌표
    
    # 로봇 모드
    ROBOT_MODE_MANUAL,
    ROBOT_MODE_AUTONOMOUS,
    ROBOT_MODE_MEASURE
)
```

---

## 8. 예제 코드 분석

### 8.1 기본 예제 - Single Robot Simple

**파일 위치:** `dsr_example2/dsr_example/dsr_example/simple/single_robot_simple.py`

```python
import rclpy
import DR_init

# 로봇 설정
ROBOT_ID = "dsr01"
ROBOT_MODEL = "m1013"

DR_init.__dsr__id = ROBOT_ID
DR_init.__dsr__model = ROBOT_MODEL

def main(args=None):
    # ROS2 초기화
    rclpy.init(args=args)
    node = rclpy.create_node('single_robot_simple_py', namespace=ROBOT_ID)
    DR_init.__dsr__node = node
    
    # DSR 라이브러리 import
    try:
        from DSR_ROBOT2 import (
            movej, movel, movec,
            set_robot_mode, set_velx, set_accx,
            ROBOT_MODE_AUTONOMOUS
        )
        from DR_common2 import posj, posx
    except ImportError as e:
        print(f"Error importing DSR_ROBOT2: {e}")
        return
    
    # 로봇 모드 설정
    set_robot_mode(ROBOT_MODE_AUTONOMOUS)
    
    # 전역 속도/가속도 설정
    set_velx(30, 20)
    set_accx(60, 40)
    
    # 위치 정의
    p1 = posj(0, 0, 0, 0, 0, 0)           # Home
    p2 = posj(0, 0, 90, 0, 90, 0)         # 목표 관절 각도
    
    x1 = posx(400, 500, 800, 0, 180, 0)   # 작업 좌표 1
    x2 = posx(400, 500, 500, 0, 180, 0)   # 작업 좌표 2
    
    # 메인 루프
    while rclpy.ok():
        # p2로 이동
        movej(p2, vel=100, acc=100)
        
        # home으로 복귀
        movej(p1, vel=100, acc=100)
    
    # 종료
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

### 8.2 실행 방법

**1단계: 로봇 실행**
```bash
# 터미널 1
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12345 \
    model:=m1013
```

**2단계: 예제 실행**
```bash
# 터미널 2
ros2 run dsr_example single_robot_simple
```

### 8.3 Pick and Place 예제

**기본 픽앤플레이스 구조:**

```python
import rclpy
import DR_init

ROBOT_ID = "dsr01"
ROBOT_MODEL = "m1013"

DR_init.__dsr__id = ROBOT_ID
DR_init.__dsr__model = ROBOT_MODEL

def main(args=None):
    rclpy.init(args=args)
    node = rclpy.create_node("pick_and_place", namespace=ROBOT_ID)
    DR_init.__dsr__node = node
    
    from DSR_ROBOT2 import movej, movel, set_tool, set_tcp
    from DR_common2 import posx, posj
    
    # Tool 설정
    set_tool("Tool Weight_2FG")
    set_tcp("2FG_TCP")
    
    # 위치 정의
    JReady = posj(0, 0, 90, 0, 90, 0)
    pick_pos = posx(400, 200, 300, 0, 180, 0)
    place_pos = posx(500, -200, 300, 0, 180, 0)
    
    vel, acc = 60, 60
    
    while rclpy.ok():
        # Ready 자세
        movej(JReady, vel=vel, acc=acc)
        
        # Pick 위치로 이동
        movel(pick_pos, vel=vel, acc=acc)
        # TODO: 그리퍼 잡기
        
        # Place 위치로 이동
        movel(place_pos, vel=vel, acc=acc)
        # TODO: 그리퍼 놓기
    
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

### 8.4 블렌딩 모션 예제

```python
from DSR_ROBOT2 import moveb, DR_LINE, DR_CIRCLE, DR_BASE, DR_MV_MOD_ABS
from DR_common2 import posx, posb

# 세그먼트 정의
X1 = posx(370, 670, 650, 0, 180, 0)
X1a = posx(370, 670, 400, 0, 180, 0)
X1a2 = posx(370, 545, 400, 0, 180, 0)
X1b2 = posx(370, 670, 400, 0, 180, 0)
X1c = posx(370, 420, 150, 0, 180, 0)
X1c2 = posx(370, 545, 150, 0, 180, 0)

# 블렌드 경로 생성
seg1 = posb(DR_LINE, X1, radius=20)
seg2 = posb(DR_CIRCLE, X1a, X1a2, radius=21)
seg3 = posb(DR_LINE, X1b2, radius=20)
seg4 = posb(DR_CIRCLE, X1c, X1c2, radius=22)

b_list = [seg1, seg2, seg3, seg4]

# 실행
moveb(b_list, vel=150, acc=250, ref=DR_BASE, mod=DR_MV_MOD_ABS)
```

---

## 9. 트러블슈팅

### 9.1 빌드 에러
### 9.1 빌드 에러

#### ❌ "Could not find DSR_ROBOT2"
```bash
# 해결: PYTHONPATH 설정
export PYTHONPATH=$PYTHONPATH:~/ros2_ws/install/dsr_common2/lib/dsr_common2/imp
source ~/.bashrc
```

#### ❌ "Package 'dsr_common2' not found"
```bash
# 해결: 패키지 재빌드
cd ~/ros2_ws
colcon build --packages-select dsr_common2
source install/setup.bash
```

### 9.2 연결 에러
#### ❌ Virtual Mode에서 "Connection refused"
```bash
# Docker가 실행 중인지 확인
sudo systemctl status docker

# 에뮬레이터 재설치
cd ~/ros2_ws/src/doosan-robot2
sudo ./uninstall_emulator.sh
sudo ./install_emulator.sh
```

#### ❌ Real Mode에서 로봇 연결 실패
1. 네트워크 연결 확인
   ```bash
   ping 192.168.137.100
   ```
2. 로봇 컨트롤러 IP/포트 확인
3. 방화벽 설정 확인

### 9.3 실행 에러

#### ❌ "Robot is not in autonomous mode"
```python
# 로봇 모드 설정 추가
from DSR_ROBOT2 import set_robot_mode, ROBOT_MODE_AUTONOMOUS
set_robot_mode(ROBOT_MODE_AUTONOMOUS)
```

#### ❌ MoveIt2 실행 안됨
- 컨트롤러 버전 2.12 이상 필요
- Real Mode에서만 사용 가능한 경우가 있음

### 9.4 성능 이슈

#### 느린 응답 속도
- Virtual Mode: Docker 리소스 증가
- Real Mode: 네트워크 대역폭 확인

#### 움직임 끊김
- 속도/가속도 파라미터 조정
- CPU 사용률 확인

---

## 9. 추가 자료

### 9.1 공식 문서
- [두산 로보틱스 ROS2 Manual](https://doosanrobotics.github.io/doosan-robotics-ros-manual/humble/index.html)
- [GitHub Repository](https://github.com/doosan-robotics/doosan-robot2)

### 9.2 유용한 명령어

```bash
# 토픽 확인
ros2 topic list

# 노드 확인
ros2 node list

# 로봇 상태 확인
ros2 topic echo /dsr01/joint_states

# 빌드 (특정 패키지만)
colcon build --packages-select dsr_example

# 빌드 (병렬 처리)
colcon build --parallel-workers 4

# 로그 레벨 설정
ros2 run dsr_example single_robot_simple --ros-args --log-level debug
```

### 9.3 디버깅 팁

1. **로봇이 움직이지 않을 때:**
   - 로봇 모드 확인 (AUTONOMOUS 여야 함)
   - Emergency Stop 상태 확인
   - 에러 메시지 확인

2. **Python import 에러:**
   - PYTHONPATH 확인
   - 패키지 빌드 확인
   - Python 버전 확인 (3.10+)

3. **Launch 파일 에러:**
   - 파라미터 철자 확인
   - 모델명 정확히 입력 (m1013, m0609 등)

---

## 10. 버전별 차이점

### Controller Version 2.x vs 3.x

```bash
# Version 2.x (기본)
colcon build

# Version 3.x
colcon build --cmake-args -DDRCF_VER=3
```

**주요 차이:**
- 3.x: 새로운 제어 알고리즘
- 3.x: 향상된 실시간 성능
- 호환성 문제 있을 수 있음

---

## 마무리

이 가이드는 두산 로보틱스 ROS2 패키지의 기본부터 고급 기능까지 다룹니다.

**학습 순서 추천:**
1. ✅ Virtual Mode + RViz2로 시작
2. ✅ single_robot_simple 예제 실행
3. ✅ 위치 정의 및 기본 모션 이해
4. ✅ 자신만의 예제 작성
5. ✅ Gazebo 시뮬레이션
6. ✅ 실제 로봇 제어 (Real Mode)

**문제 발생 시:**
- [GitHub Issues](https://github.com/doosan-robotics/doosan-robot2/issues)
- 공식 문서 확인
- 커뮤니티 포럼

---

**작성자:** GitHub Copilot  
**마지막 업데이트:** 2025년 12월 21일  
**패키지 버전:** doosan-robot2 (humble branch)
