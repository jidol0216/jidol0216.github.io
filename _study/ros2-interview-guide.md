---
title: "ROS2 면접 대비 가이드"
collection: study
type: "Guide"
permalink: /study/ros2-interview-guide
date: 2025-12-21
excerpt: "ROS2 기술 면접 완벽 대비. 25개 핵심 질문과 답변, 프로젝트 경험 어필 방법, STAR 기법 활용법."
---

# ROS2 면접 대비 가이드

## 목차
1. [ROS2 기본 개념](#ros2-기본-개념)
2. [워크스페이스와 빌드 시스템](#워크스페이스와-빌드-시스템)
3. [환경 설정 (source)](#환경-설정-source)
4. [Docker와 ROS2](#docker와-ros2)
5. [실전 면접 질문](#실전-면접-질문)

---

## ROS2 기본 개념

### Q: ROS2가 무엇인가요?
**A:** Robot Operating System 2의 약자로, 로봇 소프트웨어 개발을 위한 **미들웨어 프레임워크**입니다.

**핵심 특징:**
- **분산 시스템**: 여러 프로세스가 네트워크로 통신
- **모듈화**: 각 기능을 독립적인 노드로 개발
- **재사용성**: 표준화된 메시지로 컴포넌트 재사용
- **실시간성**: DDS 기반으로 실시간 통신 지원

### Q: ROS1과 ROS2의 차이는?
**A:** 

| 항목 | ROS1 | ROS2 |
|------|------|------|
| 통신 방식 | TCPROS (TCP/UDP) | DDS (Data Distribution Service) |
| 실시간성 | ❌ 약함 | ✅ 강함 |
| 보안 | ❌ 없음 | ✅ 암호화, 인증 지원 |
| 멀티 플랫폼 | Linux 중심 | Linux, Windows, macOS |
| Master 노드 | ✅ 필요 (roscore) | ❌ 불필요 (분산형) |
| 상용화 | 연구/프로토타입 | 산업/상용 로봇 |

### Q: Node, Topic, Service의 차이는?
**A:**

**Node (노드)**
```python
# 독립적으로 실행되는 프로세스
class MyNode(Node):
    def __init__(self):
        super().__init__('my_node_name')
```
- ROS2의 기본 실행 단위
- 각각 독립된 프로세스로 실행
- 예: 카메라 노드, 모터 제어 노드, 센서 노드

**Topic (토픽)**
```python
# 비동기 메시지 발행/구독 (Pub-Sub)
publisher = self.create_publisher(String, 'topic_name', 10)
subscriber = self.create_subscription(String, 'topic_name', callback, 10)
```
- **1:N, N:1, N:N** 통신 가능
- **비동기**: 응답 대기 없음
- 센서 데이터, 상태 정보 등에 사용
- 예: `/camera/image`, `/robot/joint_states`

**Service (서비스)**
```python
# 동기 요청-응답 (Request-Response)
client = self.create_client(AddTwoInts, 'add_two_ints')
service = self.create_service(AddTwoInts, 'add_two_ints', callback)
```
- **1:1** 동기 통신
- 요청 후 **응답 대기**
- 일시적 작업에 사용
- 예: 로봇 정지, 설정 변경

**Action (액션)**
```python
# 장시간 작업 + 중간 피드백
action_server = ActionServer(self, Fibonacci, 'fibonacci', execute_callback)
```
- 긴 작업 (예: 로봇 이동, 물체 잡기)
- **Goal** (목표), **Feedback** (중간 결과), **Result** (최종 결과)
- 중간에 취소 가능

---

## 워크스페이스와 빌드 시스템

### Q: ROS2 워크스페이스란?
**A:** ROS2 패키지들을 개발하고 빌드하는 **독립된 작업 공간**입니다.

**구조:**
```
~/ros2_ws/                  # 워크스페이스 루트
├── src/                    # 소스 코드
│   ├── package1/
│   └── package2/
├── build/                  # 빌드 중간 파일
├── install/                # 설치된 실행 파일
│   └── setup.bash         # 환경 설정 스크립트 ★
└── log/                    # 빌드 로그
```

### Q: colcon이란?
**A:** ROS2의 **빌드 도구**입니다.

**주요 명령어:**
```bash
# 전체 빌드
colcon build

# 특정 패키지만 빌드
colcon build --packages-select my_package

# 병렬 빌드 (빠름)
colcon build --parallel-workers 4

# 심볼릭 링크 (Python 패키지)
colcon build --symlink-install
```

**colcon build가 하는 일:**
1. C++ 코드 컴파일
2. Python 패키지 설치
3. `install/` 폴더에 실행 파일 생성
4. **`setup.bash` 자동 생성** ← 중요!

### Q: colcon build 후 왜 source가 필요한가?
**A:** 빌드한 패키지를 시스템이 찾을 수 있도록 **환경변수를 설정**하기 위해서입니다.

**이유:**
- ROS2는 **시스템 전역에 설치하지 않음**
- 각 워크스페이스가 **독립적**
- `source`로 경로를 환경변수에 등록

---

## 환경 설정 (source)

### Q: source 명령이 하는 일은?
**A:** `setup.bash` 스크립트를 실행하여 **환경변수를 현재 셸에 적용**합니다.

**실제 동작:**
```bash
source /home/jidol/ros2_ws/install/setup.bash
```

**설정되는 환경변수:**
```bash
# 1. ROS2 패키지 경로
export AMENT_PREFIX_PATH=/home/jidol/ros2_ws/install/package1:...

# 2. Python 패키지 경로
export PYTHONPATH=/home/jidol/ros2_ws/install/package1/lib/python3.10/site-packages:...

# 3. 실행 파일 경로
export PATH=/home/jidol/ros2_ws/install/package1/bin:$PATH

# 4. 라이브러리 경로
export LD_LIBRARY_PATH=/home/jidol/ros2_ws/install/package1/lib:...

# 5. ROS 설정
export ROS_DISTRO=humble
export ROS_VERSION=2
```

### Q: 웹개발에서는 왜 source가 필요 없나요?
**A:** 설치 방식의 차이 때문입니다.

**웹개발 (npm, pip)**
```bash
npm install -g express    # 시스템 전역에 설치
# → /usr/local/bin 등 시스템 PATH에 자동 등록
node app.js              # 바로 실행 가능
```

**ROS2 (colcon)**
```bash
colcon build             # 워크스페이스에만 설치
# → 시스템 PATH에 등록 안 됨!
source install/setup.bash  # 환경변수 등록 필요
ros2 launch ...          # 그래야 실행 가능
```

### Q: 여러 워크스페이스를 어떻게 관리하나요?
**A:** 각 워크스페이스마다 source를 따로 실행합니다.

**시나리오: 두 개의 프로젝트**
```bash
# 터미널 1 - 구버전 로봇 개발
cd ~/old_robot_ws
source install/setup.bash
ros2 launch old_robot ...

# 터미널 2 - 신버전 로봇 개발  
cd ~/new_robot_ws
source install/setup.bash
ros2 launch new_robot ...
```

**오버레이 (Overlay)**
```bash
# 기본 ROS2 로드
source /opt/ros/humble/setup.bash

# 내 워크스페이스 로드 (덮어쓰기)
source ~/ros2_ws/install/setup.bash
```
- 같은 패키지 이름이면 **나중 것이 우선**
- 기본 ROS2 위에 내 패키지 추가

### Q: 매번 source를 입력하지 않으려면?
**A:** `~/.bashrc`에 추가하면 자동 로드됩니다.

```bash
# bashrc에 추가
echo "source /home/jidol/ros2_ws/install/setup.bash" >> ~/.bashrc

# 적용
source ~/.bashrc
```

**장점:**
- 새 터미널마다 자동 설정
- 명령어 입력 생략

**단점:**
- 여러 워크스페이스 사용 시 충돌 가능
- 신중하게 사용

---

## Docker와 ROS2

### Q: ROS2에서 Docker를 왜 사용하나요?
**A:** **환경 독립성**과 **재현성**을 위해서입니다.

**장점:**
1. **일관된 환경**: 어떤 PC에서도 동일하게 동작
2. **버전 충돌 방지**: 호스트 시스템과 격리
3. **배포 용이**: 이미지 하나로 전체 환경 공유
4. **시뮬레이션**: 실제 로봇 없이 테스트 가능

**두산 로봇 예시:**
```bash
# Docker 이미지 (에뮬레이터)
docker pull doosanrobot/dsr_emulator:3.0.1

# ROS2 launch가 자동으로 Docker 실행
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual
```

### Q: Docker와 Virtual Machine의 차이는?
**A:**

| 항목 | Docker | Virtual Machine |
|------|--------|-----------------|
| 격리 수준 | 프로세스 격리 | OS 전체 격리 |
| 시작 속도 | 초 단위 | 분 단위 |
| 크기 | MB | GB |
| 오버헤드 | 낮음 | 높음 |
| 용도 | 애플리케이션 실행 | OS 전체 에뮬레이션 |

**Docker 구조:**
```
[Host OS]
├── Docker Engine
    ├── Container 1 (ROS2 Humble)
    ├── Container 2 (ROS2 Foxy)
    └── Container 3 (로봇 에뮬레이터)
```

**VM 구조:**
```
[Host OS]
├── Hypervisor
    ├── VM 1 [Guest OS + ROS2]
    ├── VM 2 [Guest OS + ROS2]
    └── VM 3 [Guest OS + 에뮬레이터]
```

### Q: Docker 컨테이너와 ROS2 통신은?
**A:** 네트워크 포트를 매핑하여 통신합니다.

```bash
docker run -d \
  --name dsr_emulator \
  -p 12345:12345 \        # 호스트:컨테이너 포트 매핑
  doosanrobot/dsr_emulator:3.0.1

# ROS2에서 연결
ros2 launch ... host:=127.0.0.1 port:=12345
```

---

## 실전 면접 질문

### 1. 기본 개념

**Q: ROS2를 사용한 프로젝트 경험이 있나요?**

예시 답변:
> "두산 로봇을 ROS2 Humble로 제어하는 프로젝트를 진행했습니다. 
> Virtual Mode에서 Docker 기반 에뮬레이터를 사용했고, 
> dsr_bringup2 패키지로 로봇을 제어했습니다.
> RViz2로 로봇 상태를 시각화하고, Python으로 trajectory 제어 코드를 작성했습니다."

**Q: colcon build와 catkin_make의 차이는?**

| 항목 | catkin_make (ROS1) | colcon (ROS2) |
|------|-------------------|--------------|
| 빌드 시스템 | CMake | CMake + Python setuptools |
| 병렬 빌드 | 제한적 | 효율적 |
| 패키지 선택 빌드 | 어려움 | 쉬움 (--packages-select) |
| Python 지원 | setup.py 필요 | 자동 인식 |

**Q: launch 파일의 역할은?**

> "여러 노드를 한 번에 실행하는 설정 파일입니다.
> ROS2에서는 Python으로 작성하며, 노드 실행, 파라미터 설정, 
> 네임스페이스 지정 등을 할 수 있습니다."

```python
# launch 파일 예시
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_package',
            executable='my_node',
            name='custom_name',
            parameters=[{'param1': 'value1'}]
        )
    ])
```

### 2. 환경 설정

**Q: source가 필요한 이유를 설명해주세요.**

> "ROS2는 워크스페이스 단위로 개발하기 때문에 
> 빌드한 패키지가 시스템 전역에 설치되지 않습니다.
> source 명령으로 워크스페이스의 경로를 환경변수에 등록해야
> ros2 명령어가 해당 패키지를 찾을 수 있습니다.
> 이를 통해 여러 프로젝트를 독립적으로 관리할 수 있습니다."

**Q: 워크스페이스 오버레이란?**

> "기본 ROS2 설치 위에 커스텀 워크스페이스를 덮어쓰는 것입니다.
> 먼저 /opt/ros/humble/setup.bash를 source하고,
> 그 다음 내 워크스페이스 setup.bash를 source하면
> 같은 이름의 패키지는 내 것이 우선 적용됩니다."

```bash
# 오버레이 순서
source /opt/ros/humble/setup.bash          # 1. 기본 ROS2
source ~/ws1/install/setup.bash            # 2. 커스텀 워크스페이스
source ~/ws2/install/setup.bash            # 3. 다른 워크스페이스
# → ws2 > ws1 > humble 순으로 우선
```

### 3. 실무 기술

**Q: ROS2에서 디버깅은 어떻게 하나요?**

```bash
# 1. 로그 레벨 조정
ros2 run package node --ros-args --log-level debug

# 2. 토픽 모니터링
ros2 topic echo /topic_name
ros2 topic hz /topic_name        # 주파수 확인

# 3. 노드 정보 확인
ros2 node list
ros2 node info /node_name

# 4. RQT 도구
rqt_graph                        # 노드 연결 시각화
rqt_console                      # 로그 확인
```

**Q: ROS2에서 파라미터를 어떻게 관리하나요?**

```bash
# 런타임에 파라미터 변경
ros2 param set /node_name param_name value

# 파라미터 목록 확인
ros2 param list /node_name

# YAML 파일로 로드
ros2 run package node --ros-args --params-file params.yaml
```

```yaml
# params.yaml
node_name:
  ros__parameters:
    param1: 10
    param2: "value"
```

**Q: DDS란 무엇인가요?**

> "Data Distribution Service의 약자로, ROS2의 통신 미들웨어입니다.
> Pub-Sub 패턴을 기반으로 실시간 데이터 통신을 지원하며,
> QoS(Quality of Service) 설정으로 신뢰성과 성능을 조절할 수 있습니다.
> FastDDS, CycloneDDS 등이 있으며 ROS2는 DDS를 추상화하여 사용합니다."

### 4. 문제 해결

**Q: "Package not found" 에러가 나면?**

체크리스트:
```bash
# 1. source 확인
echo $AMENT_PREFIX_PATH

# 2. 패키지 빌드 확인
colcon list
ls install/

# 3. 재빌드
colcon build --packages-select my_package

# 4. source 재적용
source install/setup.bash
```

**Q: 두 노드 간 통신이 안 되면?**

```bash
# 1. 노드 실행 확인
ros2 node list

# 2. 토픽 발행/구독 확인
ros2 topic list
ros2 topic info /topic_name

# 3. 메시지 타입 확인
ros2 interface show std_msgs/msg/String

# 4. ROS_DOMAIN_ID 확인 (네트워크 격리)
echo $ROS_DOMAIN_ID
```

### 5. 고급 주제

**Q: ROS2의 QoS란?**

> "Quality of Service의 약자로, 통신 품질을 설정하는 정책입니다."

**주요 QoS 설정:**
```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

qos_profile = QoSProfile(
    reliability=ReliabilityPolicy.RELIABLE,  # 신뢰성 (BEST_EFFORT / RELIABLE)
    history=HistoryPolicy.KEEP_LAST,        # 히스토리 (KEEP_ALL / KEEP_LAST)
    depth=10                                 # 큐 크기
)

self.create_subscription(String, 'topic', callback, qos_profile)
```

- **RELIABLE**: 패킷 손실 시 재전송 (중요 데이터)
- **BEST_EFFORT**: 재전송 안 함 (센서 데이터)

**Q: ROS2 보안은 어떻게 구현하나요?**

> "SROS2(Secure ROS2)를 사용하여 DDS 보안 플러그인을 적용합니다.
> 인증서 기반 암호화, 접근 제어, 암호화 통신을 지원합니다."

```bash
# 보안 키 생성
ros2 security create_keystore demo_keystore
ros2 security create_key demo_keystore /my_node

# 보안 활성화
export ROS_SECURITY_ENABLE=true
export ROS_SECURITY_KEYSTORE=/path/to/demo_keystore
```

---

## 추가 팁

### 면접 준비 체크리스트

✅ **기본 개념**
- [ ] ROS2 아키텍처 설명 가능
- [ ] Node, Topic, Service, Action 차이 이해
- [ ] DDS와 QoS 개념 이해

✅ **실습 경험**
- [ ] 최소 1개 이상 프로젝트 경험
- [ ] launch 파일 작성 경험
- [ ] 디버깅 경험 (로그, topic echo 등)

✅ **빌드 시스템**
- [ ] colcon build 사용 경험
- [ ] source 필요성 이해
- [ ] 워크스페이스 구조 이해

✅ **고급 주제**
- [ ] Docker 사용 경험
- [ ] 멀티 로봇 시스템 이해
- [ ] 보안, 성능 최적화 개념

### 프로젝트 설명 템플릿

```
[프로젝트명]
- 기간: 2024.11 ~ 2024.12 (2개월)
- 사용 기술: ROS2 Humble, Python, Docker, RViz2
- 역할: 로봇 제어 알고리즘 개발

[주요 구현 내용]
1. 두산 로봇 M1013 모델 Virtual Mode 시뮬레이션 환경 구축
2. Trajectory 제어를 위한 Python API 개발
3. RViz2를 활용한 로봇 상태 모니터링 시스템 구현
4. Docker 기반 에뮬레이터 통합

[기술적 도전]
- 문제: 로봇과 연결이 불안정
- 해결: QoS 설정 조정 및 네트워크 타임아웃 최적화
- 결과: 안정성 95% → 99.5% 향상

[배운 점]
- ROS2의 분산 아키텍처 이해
- 실시간 제어 시스템에서의 통신 최적화
- Docker를 활용한 개발 환경 표준화
```

---

## 참고 자료

### 공식 문서
- [ROS2 공식 문서](https://docs.ros.org/en/humble/)
- [ROS2 Tutorials](https://docs.ros.org/en/humble/Tutorials.html)
- [colcon 문서](https://colcon.readthedocs.io/)

### 추천 학습 자료
- ROS2 Design Patterns (Best Practices)
- DDS Specification
- ROS2 Security

### 실습 프로젝트 아이디어
1. 간단한 Pub-Sub 노드 작성
2. Service 기반 계산기 만들기
3. Action으로 긴 작업 제어하기
4. launch 파일로 멀티 노드 실행
5. Docker로 ROS2 환경 패키징

---

*최종 업데이트: 2025년 12월 21일*

