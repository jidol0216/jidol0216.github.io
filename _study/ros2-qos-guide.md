---
title: "ROS2 QoS 완벽 가이드"
collection: study
type: "Guide"
permalink: /study/ros2-qos-guide
date: 2025-12-21
---

# ROS2 QoS (Quality of Service) 완벽 가이드

## 목차
1. [QoS란 무엇인가?](#qos란-무엇인가)
2. [QoS가 중요한 이유](#qos가-중요한-이유)
3. [QoS 정책 상세 설명](#qos-정책-상세-설명)
4. [QoS 프로필](#qos-프로필)
5. [실전 예제](#실전-예제)
6. [문제 상황과 해결](#문제-상황과-해결)
7. [면접 대비 핵심 정리](#면접-대비-핵심-정리)

---

## QoS란 무엇인가?

### 정의
**QoS (Quality of Service)**는 ROS2의 통신 품질을 제어하는 정책입니다.

DDS(Data Distribution Service)의 기능을 활용하여 **데이터 전송의 신뢰성, 속도, 메모리 사용량** 등을 세밀하게 조절할 수 있습니다.

### 왜 ROS2에만 있나요?

| ROS1 | ROS2 |
|------|------|
| TCP/UDP 사용 | DDS 사용 |
| QoS 없음 | QoS 지원 ✅ |
| 통신 품질 제어 불가 | 세밀한 제어 가능 |
| 센서/제어 구분 안 됨 | 데이터 특성별 최적화 |

**ROS2는 산업용 로봇을 목표**로 설계되어 **실시간성과 안정성**이 매우 중요합니다!

---

## QoS가 중요한 이유

### 1. 데이터 특성이 다름

로봇 시스템에는 다양한 데이터가 있습니다:

```
카메라 센서 (30fps)
├─ 초당 30개 이미지
├─ 하나 손실돼도 다음 것이 곧 옴
└─ 속도가 중요! → BEST_EFFORT

로봇 제어 명령
├─ 한 번만 전송
├─ 절대 손실되면 안 됨
└─ 신뢰성이 중요! → RELIABLE

배터리 상태
├─ 천천히 변함
├─ 최신 값만 필요
└─ 큐 크기 1 → KEEP_LAST depth=1
```

### 2. 네트워크 환경이 다름

```
실험실 환경
├─ 유선 LAN
├─ 안정적
└─ RELIABLE 사용 가능

공장 현장
├─ WiFi, 간섭 많음
├─ 불안정
└─ BEST_EFFORT + 재전송 로직

우주/수중 로봇
├─ 초고지연
├─ 대역폭 제한
└─ 최소 데이터 + 압축
```

### 3. 실시간성이 중요

```python
# 잘못된 QoS: 제어 명령이 밀림
# 로봇이 0.5초 늦게 반응 → 충돌!

# 올바른 QoS: 최신 명령만 처리
# 실시간 제어 가능 ✅
```

---

## QoS 정책 상세 설명

### 1. Reliability (신뢰성) ⭐⭐⭐

**가장 중요한 설정!**

#### RELIABLE (신뢰성 보장)
```python
reliability=ReliabilityPolicy.RELIABLE
```

**특징:**
- 데이터 손실 시 **재전송**
- TCP처럼 동작
- **순서 보장**
- 네트워크 부하 증가

**사용 케이스:**
```python
# ✅ 로봇 제어 명령 (절대 손실 안 됨!)
self.cmd_pub = self.create_publisher(
    JointTrajectory,
    '/robot/joint_command',
    QoSProfile(reliability=ReliabilityPolicy.RELIABLE)
)

# ✅ 중요한 상태 정보
# - 에러 메시지
# - 안전 정지 신호
# - 배터리 경고
```

#### BEST_EFFORT (최선 노력)
```python
reliability=ReliabilityPolicy.BEST_EFFORT
```

**특징:**
- 재전송 **없음**
- UDP처럼 동작
- **빠름, 가벼움**
- 일부 데이터 손실 가능

**사용 케이스:**
```python
# ✅ 고속 센서 데이터
self.camera_sub = self.create_subscription(
    Image,
    '/camera/image_raw',
    callback,
    QoSProfile(reliability=ReliabilityPolicy.BEST_EFFORT)
)

# ✅ 고주파 데이터
# - 카메라 (30fps): 하나 손실돼도 다음 프레임 곧 옴
# - LiDAR (10Hz): 0.1초마다 새 데이터
# - IMU (100Hz): 초당 100개, 하나쯤은 괜찮음
```

### 2. Durability (영속성)

#### VOLATILE (휘발성 - 기본값)
```python
durability=DurabilityPolicy.VOLATILE
```

**특징:**
- 새 구독자는 **과거 데이터 못 받음**
- 현재 시점부터만 수신
- 메모리 효율적

**사용 케이스:**
```python
# ✅ 실시간 센서 데이터
# - 카메라 이미지
# - 레이저 스캔
# - 현재 상태
```

#### TRANSIENT_LOCAL (일시 보관)
```python
durability=DurabilityPolicy.TRANSIENT_LOCAL
```

**특징:**
- **과거 데이터 저장**
- 늦게 접속한 구독자도 받을 수 있음
- 메모리 사용량 증가

**사용 케이스:**
```python
# ✅ 설정 정보 (늦게 켜진 노드도 필요)
self.config_pub = self.create_publisher(
    RobotConfig,
    '/robot/config',
    QoSProfile(durability=DurabilityPolicy.TRANSIENT_LOCAL)
)

# ✅ 맵 데이터
# ✅ 로봇 파라미터
# ✅ 초기화 정보
```

### 3. History (히스토리)

#### KEEP_LAST (최근 N개만 보관)
```python
history=HistoryPolicy.KEEP_LAST,
depth=10  # 최근 10개
```

**특징:**
- 큐 크기 제한
- 오래된 것 자동 삭제
- **메모리 안전**

**사용 케이스:**
```python
# ✅ 대부분의 경우
# depth=1: 최신 값만 (배터리 상태)
# depth=10: 약간의 버퍼 (일반 센서)
# depth=100: 많은 버퍼 (로그)
```

#### KEEP_ALL (전부 보관)
```python
history=HistoryPolicy.KEEP_ALL
```

**특징:**
- 모든 메시지 저장
- **메모리 위험!**
- 처리 속도 < 수신 속도 → 메모리 터짐

**사용 케이스:**
```python
# ⚠️ 주의해서 사용!
# ✅ 짧은 시간 동안만 데이터 수집
# ✅ 로그 기록 (파일로 저장 후 비움)
```

### 4. Deadline (마감 시한)

```python
deadline=Duration(seconds=0, nanoseconds=100000000)  # 0.1초
```

**특징:**
- N초 안에 데이터 안 오면 **경고**
- 센서 고장 감지
- 통신 두절 감지

**사용 케이스:**
```python
# ✅ 중요 센서 모니터링
qos = QoSProfile(
    deadline=Duration(seconds=0, nanoseconds=100000000)  # 0.1초
)

# 0.1초마다 데이터 와야 함
# 안 오면 콜백 호출: on_deadline_missed()
```

### 5. Lifespan (수명)

```python
lifespan=Duration(seconds=5)  # 5초 후 폐기
```

**특징:**
- 오래된 데이터 자동 삭제
- 시간 지나면 무효

**사용 케이스:**
```python
# ✅ 센서 캘리브레이션 데이터
# ✅ 임시 명령
# ✅ 5초 이상 지나면 의미 없는 데이터
```

### 6. Liveliness (생존 확인)

```python
liveliness=LivelinessPolicy.AUTOMATIC
```

**특징:**
- 발행자가 살아있는지 확인
- 죽으면 구독자에게 알림

**정책:**
- **AUTOMATIC**: DDS가 자동 확인
- **MANUAL_BY_TOPIC**: 토픽별 수동 확인
- **MANUAL_BY_NODE**: 노드별 수동 확인

---

## QoS 프로필

### 기본 제공 프로필

ROS2는 자주 사용하는 조합을 **프리셋**으로 제공합니다.

#### 1. Sensor Data (센서 데이터)
```python
from rclpy.qos import qos_profile_sensor_data

# 설정:
# - BEST_EFFORT (빠름)
# - VOLATILE (과거 데이터 안 줌)
# - KEEP_LAST, depth=5

self.create_subscription(
    LaserScan,
    '/scan',
    callback,
    qos_profile_sensor_data  # 센서 데이터용
)
```

**적합한 데이터:**
- 카메라 이미지
- LiDAR 스캔
- IMU 데이터

#### 2. System Default (시스템 기본)
```python
from rclpy.qos import qos_profile_system_default

# 설정:
# - RELIABLE (신뢰성)
# - VOLATILE
# - KEEP_LAST, depth=10
```

**적합한 데이터:**
- 일반적인 토픽
- 중요도 중간

#### 3. Services (서비스 기본)
```python
from rclpy.qos import qos_profile_services_default

# 설정:
# - RELIABLE (반드시 전달)
# - VOLATILE
# - KEEP_LAST, depth=10
```

#### 4. Parameters (파라미터)
```python
from rclpy.qos import qos_profile_parameters

# 설정:
# - RELIABLE
# - TRANSIENT_LOCAL (늦게 켜진 노드도 받음)
# - KEEP_LAST, depth=1000
```

### 커스텀 프로필 만들기

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy, DurabilityPolicy

# 로봇 제어 명령용 (절대 손실 안 됨!)
control_qos = QoSProfile(
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.VOLATILE,
    history=HistoryPolicy.KEEP_LAST,
    depth=1  # 최신 명령만
)

# 고속 카메라용 (속도 중요)
camera_qos = QoSProfile(
    reliability=ReliabilityPolicy.BEST_EFFORT,
    durability=DurabilityPolicy.VOLATILE,
    history=HistoryPolicy.KEEP_LAST,
    depth=1  # 최신 프레임만
)

# 중요 로그용 (전부 저장)
log_qos = QoSProfile(
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.TRANSIENT_LOCAL,
    history=HistoryPolicy.KEEP_LAST,
    depth=1000  # 많이 저장
)
```

---

## 실전 예제

### 예제 1: 카메라 노드 (고속 센서)

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

class CameraNode(Node):
    def __init__(self):
        super().__init__('camera_node')
        
        # 카메라용 QoS (속도 최우선)
        camera_qos = QoSProfile(
            reliability=ReliabilityPolicy.BEST_EFFORT,  # 재전송 안 함
            history=HistoryPolicy.KEEP_LAST,
            depth=1  # 최신 프레임만
        )
        
        self.publisher = self.create_publisher(
            Image,
            '/camera/image_raw',
            camera_qos
        )
        
        self.timer = self.create_timer(0.033, self.publish_image)  # 30fps
        
    def publish_image(self):
        msg = Image()
        # ... 이미지 데이터 채우기
        self.publisher.publish(msg)
        # 손실돼도 괜찮음, 0.033초 후 새 이미지 옴!
```

### 예제 2: 로봇 제어 노드 (신뢰성 필수)

```python
import rclpy
from rclpy.node import Node
from trajectory_msgs.msg import JointTrajectory
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

class RobotControlNode(Node):
    def __init__(self):
        super().__init__('robot_control_node')
        
        # 제어 명령용 QoS (신뢰성 최우선)
        control_qos = QoSProfile(
            reliability=ReliabilityPolicy.RELIABLE,  # 재전송 보장!
            history=HistoryPolicy.KEEP_LAST,
            depth=1  # 최신 명령만 (오래된 명령 무의미)
        )
        
        self.publisher = self.create_publisher(
            JointTrajectory,
            '/robot/joint_command',
            control_qos
        )
        
    def send_command(self, positions):
        msg = JointTrajectory()
        # ... 명령 채우기
        self.publisher.publish(msg)
        # RELIABLE이므로 반드시 전달됨!
```

### 예제 3: 설정 노드 (늦게 켜진 노드도 받음)

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy, HistoryPolicy

class ConfigNode(Node):
    def __init__(self):
        super().__init__('config_node')
        
        # 설정 정보용 QoS
        config_qos = QoSProfile(
            reliability=ReliabilityPolicy.RELIABLE,
            durability=DurabilityPolicy.TRANSIENT_LOCAL,  # 과거 데이터 보관!
            history=HistoryPolicy.KEEP_LAST,
            depth=10
        )
        
        self.publisher = self.create_publisher(
            String,
            '/robot/config',
            config_qos
        )
        
        # 초기 설정 발행
        self.publish_config()
        
    def publish_config(self):
        msg = String()
        msg.data = "robot_speed=1.0;mode=auto"
        self.publisher.publish(msg)
        # 나중에 켜진 노드도 이 메시지 받음!
```

### 예제 4: 안전 모니터링 (Deadline 사용)

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from rclpy.qos import QoSProfile, ReliabilityPolicy, Duration
from rclpy.event_handler import SubscriptionEventCallbacks
from rclpy.qos_event import QoSRequestedDeadlineMissedInfo

class SafetyMonitor(Node):
    def __init__(self):
        super().__init__('safety_monitor')
        
        # 0.1초 안에 데이터 와야 함!
        safety_qos = QoSProfile(
            reliability=ReliabilityPolicy.RELIABLE,
            deadline=Duration(seconds=0, nanoseconds=100000000)  # 0.1초
        )
        
        # 이벤트 콜백 설정
        callbacks = SubscriptionEventCallbacks(
            deadline=self.deadline_callback
        )
        
        self.subscription = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            safety_qos,
            event_callbacks=callbacks
        )
        
    def scan_callback(self, msg):
        self.get_logger().info('센서 정상')
        # 장애물 감지 로직
        
    def deadline_callback(self, event: QoSRequestedDeadlineMissedInfo):
        self.get_logger().error('⚠️ 센서 응답 없음! 로봇 정지!')
        # 비상 정지 로직
        self.emergency_stop()
```

---

## 문제 상황과 해결

### 문제 1: "메시지가 안 와요!"

#### 증상
```python
# 발행자
self.create_publisher(String, '/topic', 10)

# 구독자
self.create_subscription(String, '/topic', callback, 10)

# 구독자가 메시지를 못 받음!
```

#### 원인
**QoS 불일치!**

```bash
# 발행자 확인
ros2 topic info /topic -v

Publishers:
  QoS Profile:
    Reliability: RELIABLE
    Durability: VOLATILE

# 구독자 확인
Subscriptions:
  QoS Profile:
    Reliability: BEST_EFFORT  # ← 다름!
    Durability: VOLATILE
```

#### 해결
**QoS를 일치**시키거나 **호환 가능**하게:

```python
# 방법 1: 둘 다 RELIABLE로
pub_qos = QoSProfile(reliability=ReliabilityPolicy.RELIABLE)
sub_qos = QoSProfile(reliability=ReliabilityPolicy.RELIABLE)

# 방법 2: 둘 다 BEST_EFFORT로
pub_qos = QoSProfile(reliability=ReliabilityPolicy.BEST_EFFORT)
sub_qos = QoSProfile(reliability=ReliabilityPolicy.BEST_EFFORT)
```

**호환성 규칙:**
| 발행자 | 구독자 | 호환 여부 |
|--------|--------|----------|
| RELIABLE | RELIABLE | ✅ |
| RELIABLE | BEST_EFFORT | ✅ |
| BEST_EFFORT | RELIABLE | ❌ |
| BEST_EFFORT | BEST_EFFORT | ✅ |

### 문제 2: "메시지가 밀려요!"

#### 증상
```python
# 초당 100개 발행
# 하지만 구독자가 초당 10개만 처리
# → 메모리 증가, 지연 발생!
```

#### 원인
**depth가 너무 큼!**

```python
# 잘못된 설정
qos = QoSProfile(
    history=HistoryPolicy.KEEP_LAST,
    depth=1000  # 너무 많이 보관!
)
```

#### 해결
```python
# 실시간 제어는 최신 값만!
qos = QoSProfile(
    history=HistoryPolicy.KEEP_LAST,
    depth=1  # 최신 것만
)

# 또는 처리 속도 높이기
def callback(self, msg):
    # 빠르게 처리
    pass
```

### 문제 3: "늦게 켠 노드가 설정을 못 받아요!"

#### 증상
```python
# 설정 발행 노드가 먼저 실행
config_pub.publish(config_msg)

# 10초 후 구독 노드 실행
# → 설정 못 받음!
```

#### 원인
**VOLATILE (기본값)** 사용

#### 해결
```python
# TRANSIENT_LOCAL 사용
config_qos = QoSProfile(
    durability=DurabilityPolicy.TRANSIENT_LOCAL  # 과거 데이터 보관!
)

publisher = self.create_publisher(
    String,
    '/config',
    config_qos
)
```

### 문제 4: "센서가 고장났는지 모르겠어요!"

#### 해결
**Deadline 사용!**

```python
from rclpy.qos import Duration

qos = QoSProfile(
    deadline=Duration(seconds=1)  # 1초 안에 와야 함
)

callbacks = SubscriptionEventCallbacks(
    deadline=lambda event: self.get_logger().error('센서 고장!')
)

self.create_subscription(
    LaserScan,
    '/scan',
    callback,
    qos,
    event_callbacks=callbacks
)
```

---

## 면접 대비 핵심 정리

### Q1: QoS가 뭔가요?

> "ROS2의 통신 품질을 제어하는 정책입니다.
> DDS를 기반으로 신뢰성, 속도, 메모리 사용량을 조절할 수 있습니다.
> 센서 데이터는 BEST_EFFORT로 빠르게, 제어 명령은 RELIABLE로 안정적으로 전송하는 등
> 데이터 특성에 맞게 최적화할 수 있습니다."

### Q2: RELIABLE과 BEST_EFFORT의 차이는?

> "RELIABLE은 TCP처럼 재전송을 통해 데이터 손실을 방지하고,
> BEST_EFFORT는 UDP처럼 재전송 없이 빠르게 전송합니다.
> 로봇 제어 명령처럼 손실되면 안 되는 데이터는 RELIABLE,
> 카메라처럼 고속으로 새 데이터가 계속 오는 경우는 BEST_EFFORT를 사용합니다."

### Q3: QoS 불일치 문제를 겪은 적 있나요?

> "네, 센서 노드와 처리 노드 간 통신이 안 되는 문제가 있었습니다.
> ros2 topic info로 확인해보니 발행자는 BEST_EFFORT, 구독자는 RELIABLE로 설정되어
> 호환되지 않았습니다. 둘 다 BEST_EFFORT로 통일하여 해결했습니다."

### Q4: depth를 어떻게 설정하나요?

> "데이터 특성에 따라 다릅니다.
> 실시간 제어는 depth=1로 최신 값만 사용하고,
> 센서 데이터는 depth=10 정도로 약간의 버퍼를 두며,
> 로그 기록은 depth=100 이상으로 많이 저장합니다.
> 중요한 것은 처리 속도보다 depth가 크면 메모리 문제가 생긴다는 점입니다."

### Q5: TRANSIENT_LOCAL은 언제 사용하나요?

> "늦게 실행된 노드도 과거 데이터를 받아야 할 때 사용합니다.
> 예를 들어 로봇 설정 정보나 맵 데이터는 한 번만 발행되는데,
> 나중에 켜진 노드도 이 정보가 필요하므로 TRANSIENT_LOCAL을 사용합니다."

### Q6: Deadline은 왜 사용하나요?

> "중요한 센서가 고장나거나 통신이 끊겼을 때 감지하기 위해 사용합니다.
> 예를 들어 LiDAR는 0.1초마다 데이터를 보내야 하는데,
> deadline을 0.1초로 설정하면 데이터가 안 올 때 콜백으로 알림을 받아
> 비상 정지 등의 안전 로직을 실행할 수 있습니다."

### Q7: QoS 프로필을 직접 만든 경험이 있나요?

> "두산 로봇 프로젝트에서 trajectory 제어용 커스텀 QoS를 만들었습니다.
> RELIABLE로 신뢰성을 보장하되, depth=1로 최신 명령만 처리하도록 했습니다.
> 이렇게 하면 네트워크 지연 시 오래된 명령이 쌓이지 않고
> 항상 최신 명령만 실행되어 실시간성이 향상되었습니다."

---

## 실전 체크리스트

### QoS 설정 전 질문

✅ **이 데이터는 손실되면 안 되나요?**
- YES → RELIABLE
- NO → BEST_EFFORT

✅ **얼마나 자주 새 데이터가 오나요?**
- 빠름 (>10Hz) → depth=1~5
- 보통 → depth=10
- 느림 → depth=100

✅ **늦게 켠 노드도 받아야 하나요?**
- YES → TRANSIENT_LOCAL
- NO → VOLATILE

✅ **데이터가 안 오면 알아야 하나요?**
- YES → Deadline 설정
- NO → 설정 안 함

✅ **오래된 데이터는 의미가 있나요?**
- NO → Lifespan 설정
- YES → 설정 안 함

### 디버깅 명령어

```bash
# QoS 확인
ros2 topic info /topic_name -v

# 발행자/구독자 QoS 비교
ros2 topic info /topic_name --verbose

# 실시간 모니터링
ros2 topic hz /topic_name    # 주파수
ros2 topic bw /topic_name    # 대역폭
```

---

## 추가 자료

### 공식 문서
- [ROS2 QoS 정책](https://docs.ros.org/en/humble/Concepts/About-Quality-of-Service-Settings.html)
- [DDS QoS 사양](https://www.omg.org/spec/DDS/)

### 추천 실습
1. 카메라 노드 QoS 최적화
2. 제어 명령 QoS 설계
3. QoS 불일치 문제 해결
4. Deadline으로 센서 모니터링

### 성능 측정
```python
# 처리 시간 측정
import time

def callback(self, msg):
    start = time.time()
    # 처리
    elapsed = time.time() - start
    self.get_logger().info(f'처리 시간: {elapsed*1000:.2f}ms')
```

---

## 요약

### 핵심 원칙

1. **센서는 BEST_EFFORT** (빠름)
2. **제어는 RELIABLE** (안전)
3. **실시간은 depth=1** (최신만)
4. **설정은 TRANSIENT_LOCAL** (과거 보관)
5. **안전은 Deadline** (고장 감지)

### QoS는 Trade-off

```
신뢰성 ↑  →  속도 ↓
메모리 ↑  →  안전 ↑
depth ↑   →  지연 ↑
```

**최적의 QoS = 데이터 특성 + 시스템 요구사항**

---

*최종 업데이트: 2025년 12월 21일*

