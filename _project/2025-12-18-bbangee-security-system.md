---
title: "BBANGEE - Military Security Robot System"
short_title: "Security Robot"
date: 2025-12-18
collection: project
permalink: /projects/bbangee-security-system/
excerpt: "Doosan M0609 로봇 팔과 RealSense 카메라를 활용한 군 초병 로봇 시스템. 얼굴 추적, IFF 피아식별, 음성 인증을 통합한 무인 경계 솔루션"
---

# 🪖 BBANGEE - Military Security Robot System

> **프로젝트 이름**: BBANGEE (빵이)  
> **기간**: 2024-2025  
> **플랫폼**: ROS2 Humble · Doosan M0609 · RealSense D435 · FastAPI · React  
> **GitHub**: [Rokey-C1/bbangee](https://github.com/Rokey-C1/bbangee)

---

## 🎯 프로젝트 개요

**BBANGEE**는 군 초병 임무를 수행하는 자율 경계 로봇 시스템입니다. Doosan 로봇 팔(M0609)이 RealSense 카메라와 OnRobot RG2 그리퍼를 장착하여 접근자를 감지하고, 얼굴 추적, IFF(Identify Friend or Foe) 피아식별, 음성 암구호 검증을 통해 아군/적군을 판별합니다.

### 핵심 기능
- 👤 **실시간 얼굴 추적**: YOLOv8 기반 얼굴 검출 및 EKF 필터링
- 🎖️ **IFF 피아식별**: 완장 검출 + OCR 텍스트 인식 (통일/멸공)
- 🎙️ **음성 인증**: 암구호 Challenge-Response 시스템
- 🤖 **자율 대응**: 상태 머신 기반 시나리오 자동 실행
- 🌐 **웹 제어**: React 기반 실시간 모니터링 및 제어
- 🔧 **충돌 복구**: 자동 충돌 감지 및 복구 모션

---

## 🏗️ 시스템 아키텍처

### 전체 구조
```
┌────────────────────────────────────────────────────────────────┐
│                   BBANGEE Security System                       │
├─────────────┬──────────────────┬───────────────────────────────┤
│  Frontend   │     Backend      │          ROS2 Layer          │
│  (React)    │    (FastAPI)     │   (Robot Control & Vision)   │
├─────────────┼──────────────────┼───────────────────────────────┤
│ • Vite      │ • Python 3.10    │ • Doosan M0609 Driver        │
│ • TypeScript│ • SQLite DB      │ • RealSense D435             │
│ • WebSocket │ • REST API       │ • Face Detection (YOLOv8)    │
│ • Camera    │ • CORS           │ • Face Tracking (EKF)        │
│   Stream    │ • ROS Bridge     │ • Joint Tracking Control     │
│ • Personnel │ • Scenario FSM   │ • IFF Armband Detection      │
│   Management│ • Voice Auth     │ • Gripper Control (Modbus)   │
│             │ • Access Log     │ • Collision Recovery         │
└─────────────┴──────────────────┴───────────────────────────────┘
         │              │                     │
         └──────────────┴─────────────────────┘
                        │
                  WebSocket + REST API
```

### 디렉토리 구조
```
bbangee/
├── bbangee/
│   ├── backend/                    # FastAPI 백엔드
│   │   └── app/
│   │       ├── routers/
│   │       │   ├── people.py       # 인원 관리
│   │       │   ├── access.py       # 출입 기록
│   │       │   ├── robot.py        # 로봇 모션 제어
│   │       │   ├── scenario.py     # 시나리오 FSM
│   │       │   ├── voice.py        # 음성 인증
│   │       │   ├── ros2.py         # ROS2 브릿지
│   │       │   ├── gripper.py      # 그리퍼 제어
│   │       │   └── pistol_grip.py  # 권총 그립
│   │       ├── models.py           # SQLAlchemy 모델
│   │       ├── schemas.py          # Pydantic 스키마
│   │       └── main.py             # FastAPI 앱
│   │
│   └── frontend/                   # React 프론트엔드
│       └── src/
│           ├── pages/
│           │   ├── DashboardPage   # 대시보드
│           │   ├── PersonnelPage   # 인원 관리
│           │   ├── AccessLogPage   # 출입 기록
│           │   └── RobotControlPage # 로봇 제어
│           └── api/                # API 클라이언트
│
├── rokey_packages/                 # ROS2 커스텀 패키지
│   ├── face_tracking/              # 🔧 얼굴 추적
│   │   └── face_tracking/
│   │       ├── detection/
│   │       │   ├── yolo_detector.py       # YOLOv8 얼굴 검출
│   │       │   └── face_detection_node.py # ROS2 노드
│   │       ├── tracking/
│   │       │   ├── face_tracking_node.py  # EKF 필터링
│   │       │   └── ekf_filter.py          # Extended Kalman Filter
│   │       └── control/
│   │           └── joint_tracking_node.py # 관절 제어
│   │
│   ├── ros2_web_bridge/            # 🔧 웹 브릿지
│   │   └── ros2_web_bridge/
│   │       ├── bridge_node.py             # WebSocket 브릿지
│   │       ├── robot_controller.py        # 로봇 명령 실행
│   │       ├── camera_streamer.py         # 카메라 스트리밍
│   │       ├── collision_recovery_node.py # 충돌 복구
│   │       └── pistol_grip_node.py        # 권총 그립 제어
│   │
│   ├── gripper_rviz_sync/          # 🔧 그리퍼 동기화
│   │   └── gripper_rviz_sync/
│   │       ├── gripper_controller.py      # Modbus 제어
│   │       ├── gripper_state_publisher.py # 상태 퍼블리시
│   │       └── joint_state_merger.py      # Joint State 병합
│   │
│   ├── camera_utils/               # 🔧 카메라 유틸
│   │   └── camera_utils/
│   │       └── image_flip_node.py         # 이미지 플립
│   │
│   ├── voice_auth/                 # 🔧 음성 인증
│   │   └── voice_auth/
│   │       ├── voice_auth_node.py         # 음성 인증 노드
│   │       └── tts_engine.py              # TTS 엔진
│   │
│   └── sam3_grip_detection/        # 🔧 그립 검출
│       └── sam3_grip_detection/
│           └── grip_detection_node.py     # SAM3 기반 검출
│
├── collision_recovery/             # 충돌 복구 모듈
│   ├── main.py                     # 복구 메인 로직
│   ├── recovery.py                 # 복구 알고리즘
│   └── robot_state.py              # 상태 모니터링
│
├── start_all.sh                    # 전체 시스템 실행 스크립트
├── start_hardware.sh               # 하드웨어 시작
├── start_ros2_nodes.sh             # ROS2 노드 시작
└── start_web.sh                    # 웹 서버 시작
```

---

## 🚀 주요 기능

### 1. 얼굴 추적 시스템

**기술 스택**:
- YOLOv8: 실시간 얼굴 검출 (640x480 @ 30fps)
- EKF (Extended Kalman Filter): 노이즈 제거 및 평활화
- ROS2 Control: 로봇 관절 제어

**Face Detection Node**:
```python
# face_detection_node.py
class FaceDetectionNode(Node):
    def __init__(self):
        super().__init__('face_detection_node')
        
        # YOLO 검출기 초기화
        self.detector = YoloDetector(
            model_path='yolov8n-face.pt',
            conf_threshold=0.5
        )
        
        # 구독자: RealSense 이미지
        self.create_subscription(
            Image,
            '/camera/camera/color/image_raw',
            self.image_callback,
            10
        )
        
        # 퍼블리셔: 얼굴 좌표
        self.face_pub = self.create_publisher(
            Float32MultiArray,
            '/face_detection/faces',
            10
        )
        
        # IFF 필터 (계층적 베이지안)
        self.iff_filter = HierarchicalIFFFilter(
            band_threshold=0.75,     # 완장 검출 임계값
            text_threshold=0.70,     # 텍스트 인식 임계값
            decay=0.92,              # 시간 감쇠
            learning_rate=0.35       # 학습률
        )
    
    def image_callback(self, msg):
        """얼굴 검출 및 IFF 판별"""
        cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        
        # YOLOv8 얼굴 검출
        detections = self.detector.detect(cv_image)
        
        if detections:
            # 가장 큰 얼굴 선택
            largest = max(detections, key=lambda d: d.width * d.height)
            
            # 좌표 퍼블리시 [center_x, center_y, width, height]
            face_data = Float32MultiArray()
            face_data.data = [
                largest.center_x,
                largest.center_y,
                largest.width,
                largest.height
            ]
            self.face_pub.publish(face_data)
            
            # IFF 완장 검출 (ROI 확장)
            self.detect_armband(cv_image, largest)
```

**EKF Tracking**:
```python
# face_tracking_node.py
class FaceTrackingNode(Node):
    def __init__(self):
        super().__init__('face_tracking_node')
        
        # EKF 필터 초기화
        self.ekf = EKFFilter(
            process_noise=0.1,    # 프로세스 노이즈
            measurement_noise=5.0 # 측정 노이즈
        )
        
        # 추적 상태
        self.tracking_active = False
        self.last_detection_time = None
        
    def face_callback(self, msg):
        """얼굴 좌표 EKF 필터링"""
        face_x, face_y, w, h = msg.data
        
        # EKF 업데이트
        filtered_x, filtered_y = self.ekf.update([face_x, face_y])
        
        # 중심 오프셋 계산 (이미지 중심 기준)
        image_center_x = 320  # 640x480 중심
        image_center_y = 240
        
        offset_x = filtered_x - image_center_x
        offset_y = filtered_y - image_center_y
        
        # 제어 명령 퍼블리시
        self.publish_tracking_command(offset_x, offset_y)
```

**Joint Tracking Control**:
```python
# joint_tracking_node.py
class JointTrackingNode(Node):
    def __init__(self):
        super().__init__('joint_tracking_node')
        
        # PID 제어기
        self.pid_pan = PIDController(kp=0.015, ki=0.001, kd=0.005)
        self.pid_tilt = PIDController(kp=0.012, ki=0.001, kd=0.004)
        
        # 관절 제한
        self.joint_limits = {
            'J1': (-180, 180),   # Pan (좌우)
            'J2': (-95, 95)      # Tilt (상하)
        }
        
    def tracking_callback(self, msg):
        """얼굴 오프셋을 관절 각도로 변환"""
        offset_x, offset_y = msg.data
        
        # PID 계산
        delta_j1 = self.pid_pan.compute(offset_x)
        delta_j2 = self.pid_tilt.compute(offset_y)
        
        # 현재 조인트 각도에 더하기
        target_j1 = self.current_joints[0] + delta_j1
        target_j2 = self.current_joints[1] + delta_j2
        
        # 제한 적용
        target_j1 = np.clip(target_j1, *self.joint_limits['J1'])
        target_j2 = np.clip(target_j2, *self.joint_limits['J2'])
        
        # MoveJ 명령 전송
        self.send_movej_command([target_j1, target_j2, ...])
```

---

### 2. IFF 피아식별 시스템

**계층적 베이지안 필터**:
- **Stage 1**: 완장 유무 판별 (NO_BAND vs BAND)
- **Stage 2**: 텍스트 분류 (TONGIL vs MELGONG)
- **결과**: ALLY (아군) / ENEMY (적군) / UNKNOWN

**알고리즘**:
```python
# HierarchicalIFFFilter (face_detection_node.py)
class HierarchicalIFFFilter:
    def __init__(self, band_threshold=0.75, text_threshold=0.70):
        self.band_threshold = band_threshold
        self.text_threshold = text_threshold
        
        # Stage 1: 완장 유무 신념 [NO_BAND, BAND]
        self.band_belief = np.array([0.5, 0.5])
        
        # Stage 2: 텍스트 분류 신념 [TONGIL, MELGONG]
        self.text_belief = np.array([0.5, 0.5])
        
        self.band_confirmed = False
        self.confirmed_state = None
    
    def update(self, stage1_prob, stage2_prob):
        """
        2단계 베이지안 업데이트
        
        Args:
            stage1_prob: {'no_band': float, 'has_band': float}
            stage2_prob: {'tongil': float, 'melgong': float}
        
        Returns:
            {'state': 'ALLY'|'ENEMY'|'UNKNOWN', 'confidence': float}
        """
        # Stage 1: 완장 검출
        obs_band = np.array([stage1_prob['no_band'], stage1_prob['has_band']])
        
        # 베이지안 업데이트 (시간 감쇠 + 새 관측)
        self.band_belief = self.decay * self.band_belief + \
                           self.learning_rate * obs_band
        self.band_belief /= self.band_belief.sum()  # 정규화
        
        # 완장 확정 여부 체크
        if self.band_belief[1] > self.band_threshold:
            self.band_confirmed = True
        
        # Stage 2: 텍스트 인식 (완장 확정된 경우만)
        if self.band_confirmed:
            obs_text = np.array([stage2_prob['tongil'], stage2_prob['melgong']])
            
            self.text_belief = self.decay * self.text_belief + \
                               self.learning_rate * obs_text
            self.text_belief /= self.text_belief.sum()
            
            # 최종 판별
            if self.text_belief[0] > self.text_threshold:
                self.confirmed_state = 'ALLY'
                return {'state': 'ALLY', 'confidence': self.text_belief[0]}
            elif self.text_belief[1] > self.text_threshold:
                self.confirmed_state = 'ENEMY'
                return {'state': 'ENEMY', 'confidence': self.text_belief[1]}
        
        return {'state': 'UNKNOWN', 'confidence': 0.0}
```

**완장 검출 프로세스**:
1. 얼굴 검출 후 ROI 확장 (어깨/팔뚝 영역)
2. CNN Stage 1: 완장 유무 이진 분류
3. 신뢰도 75% 이상 시 Stage 2 활성화
4. CNN Stage 2: OCR 텍스트 인식 (통일/멸공)
5. 신뢰도 70% 이상 시 최종 판별

---

### 3. 시나리오 상태 머신

**상태 흐름**:
```
IDLE (경계)
  ↓ 얼굴 검출
DETECTED (접근자 감지)
  ↓ "멈춰!"
IDENTIFY (피아식별 대기)
  ↓ IFF 판별 완료
  ├─ ALLY → PASSWORD_CHECK (암구호 검증)
  │    ↓ 암구호 정답
  │    ├─ ALLY_PASS (통과 승인)
  │    │    └─ 차단봉 열기, "통과하십시오"
  │    └─ 암구호 오답
  │         └─ ALLY_ALERT (아군 경고)
  │              └─ "암구호 오답입니다"
  │
  └─ ENEMY → PASSWORD_CHECK
       ↓ 암구호 정답
       ├─ ENEMY_CRITICAL (기밀 유출)
       │    └─ "기밀이 유출되었습니다!"
       └─ 암구호 오답
            └─ ENEMY_ENGAGE (적군 대응)
                 └─ 경계 자세, "즉시 떠나십시오!"
```

**Scenario Manager**:
```python
# scenario.py
class ScenarioManager:
    def __init__(self):
        self.state = ScenarioState.IDLE
        self.person_type = PersonType.UNKNOWN
        self.password_challenge = "로키"   # 질문
        self.password_response = "협동"    # 정답
        
    async def handle_face_detected(self):
        """얼굴 감지 이벤트"""
        if self.state == ScenarioState.IDLE:
            self.state = ScenarioState.DETECTED
            self.detection_time = datetime.now()
            
            # TTS: "멈춰! 신원을 확인하겠습니다."
            await self.play_tts("멈춰! 신원을 확인하겠습니다.")
            
            await asyncio.sleep(0.3)
            self.state = ScenarioState.IDENTIFY
            
            # 로봇 모션: High Ready 자세
            await self.robot_motion("high_ready")
            
            return {"state": self.state, "message": "접근자 감지"}
    
    async def handle_iff_result(self, person_type: str):
        """IFF 피아식별 결과"""
        if self.state == ScenarioState.IDENTIFY:
            self.person_type = PersonType(person_type)
            
            if person_type == "ALLY":
                await self.play_tts("아군으로 확인되었습니다.")
            elif person_type == "ENEMY":
                await self.play_tts("적군으로 확인되었습니다.")
            
            await asyncio.sleep(5.0)
            self.state = ScenarioState.PASSWORD_CHECK
            
            # 암구호 질문
            await self.play_tts(f"암구호를 대십시오. {self.password_challenge}?")
            
            return {"state": self.state, "person_type": person_type}
    
    async def handle_password(self, user_answer: str):
        """암구호 검증"""
        if self.state != ScenarioState.PASSWORD_CHECK:
            return {"error": "암구호 검증 상태가 아닙니다"}
        
        is_correct = (user_answer.strip() == self.password_response)
        
        # 4가지 시나리오
        if self.person_type == PersonType.ALLY and is_correct:
            self.state = ScenarioState.ALLY_PASS
            await self.play_tts("통과하십시오.")
            await self.robot_motion("barrier_open")
            await asyncio.sleep(5.0)
            self.reset()
            
        elif self.person_type == PersonType.ALLY and not is_correct:
            self.state = ScenarioState.ALLY_ALERT
            await self.play_tts("암구호가 틀렸습니다. 재확인하십시오.")
            await asyncio.sleep(3.0)
            self.reset()
            
        elif self.person_type == PersonType.ENEMY and is_correct:
            self.state = ScenarioState.ENEMY_CRITICAL
            await self.play_tts("경고! 기밀이 유출되었습니다!")
            await self.robot_motion("threat")
            await asyncio.sleep(5.0)
            self.reset()
            
        elif self.person_type == PersonType.ENEMY and not is_correct:
            self.state = ScenarioState.ENEMY_ENGAGE
            await self.play_tts("즉시 이 구역을 떠나십시오!")
            await self.robot_motion("threat")
            await asyncio.sleep(5.0)
            self.reset()
        
        return {"state": self.state, "result": is_correct}
    
    def reset(self):
        """초기 상태로 리셋"""
        self.state = ScenarioState.IDLE
        self.person_type = PersonType.UNKNOWN
        self.detection_time = None
        self.ocr_ally_count = 0
        self.ocr_enemy_count = 0
```

**API 엔드포인트**:
```python
@router.post("/password")
async def check_password(request: PasswordRequest):
    """암구호 검증"""
    result = await scenario_manager.handle_password(request.answer)
    
    # 인원 관리 DB 업데이트
    if result['state'] == ScenarioState.ALLY_PASS:
        db.record_access_entry(military_serial=request.serial)
    
    return result

@router.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """실시간 상태 스트리밍"""
    await websocket.accept()
    scenario_manager.websockets.append(websocket)
    
    try:
        while True:
            # 클라이언트로부터 명령 수신
            data = await websocket.receive_json()
            
            if data['type'] == 'face_detected':
                result = await scenario_manager.handle_face_detected()
                await websocket.send_json(result)
            
            elif data['type'] == 'iff_result':
                result = await scenario_manager.handle_iff_result(data['person_type'])
                await websocket.send_json(result)
    
    except WebSocketDisconnect:
        scenario_manager.websockets.remove(websocket)
```

---

### 4. 로봇 모션 제어

**미리 정의된 모션**:
```python
# robot.py
MOTIONS = {
    "salute": {
        "name": "경례",
        "joints": [3.0, 0.0, 60.0, 120.0, 45.0, 0.0],
        "velocity": 25.0,
        "acceleration": 20.0,
    },
    "high_ready": {
        "name": "High Ready",
        "joints": [3.0, -20.0, 92.0, 86.0, 0.0, 0.0],
        "velocity": 30.0,
        "acceleration": 25.0,
    },
    "home": {
        "name": "홈",
        "joints": [0.0, 0.0, 90.0, 0.0, 90.0, 0.0],
        "velocity": 30.0,
        "acceleration": 25.0,
    },
    "barrier_open": {
        "name": "차단봉 열기",
        "joints": [3.0, -15.0, 80.0, 100.0, -20.0, 0.0],
        "velocity": 20.0,
        "acceleration": 15.0,
    },
    "threat": {
        "name": "위협",
        "joints": [35.0, -20.0, 110.0, 50.0, 10.0, 0.0],
        "velocity": 40.0,
        "acceleration": 35.0,
    },
}

@router.post("/motion")
def execute_motion(request: MotionRequest):
    """미리 정의된 모션 실행"""
    if request.motion not in MOTIONS:
        raise HTTPException(404, "모션을 찾을 수 없습니다")
    
    motion = MOTIONS[request.motion]
    
    # 명령 파일 작성 (ROS2 Bridge가 읽음)
    command = {
        "type": "movej",
        "joints": motion["joints"],
        "velocity": motion["velocity"],
        "acceleration": motion["acceleration"],
        "timestamp": time.time()
    }
    
    with open(COMMAND_FILE, 'w') as f:
        json.dump(command, f)
    
    return {"success": True, "motion": motion["name"]}
```

**Robot Controller Node**:
```python
# robot_controller.py
class RobotControllerNode(Node):
    def __init__(self):
        super().__init__('robot_controller')
        
        # Doosan SDK 초기화
        self.set_robot_mode(ROBOT_MODE_MANUAL)
        self.set_robot_system(ROBOT_SYSTEM_REAL)
        
        # 명령 파일 모니터링 (0.1초 주기)
        self.create_timer(0.1, self.check_command_file)
        
    def check_command_file(self):
        """명령 파일 읽기 및 실행"""
        if not os.path.exists(COMMAND_FILE):
            return
        
        try:
            with open(COMMAND_FILE, 'r') as f:
                command = json.load(f)
            
            if command['type'] == 'movej':
                self.execute_movej(
                    command['joints'],
                    command['velocity'],
                    command['acceleration']
                )
            
            # 실행 후 파일 삭제
            os.remove(COMMAND_FILE)
            
        except Exception as e:
            self.get_logger().error(f"명령 실행 실패: {e}")
    
    def execute_movej(self, joints, vel, acc):
        """MoveJ 명령 실행"""
        DR_MoveJ(joints, vel=vel, acc=acc)
        self.get_logger().info(f"MoveJ 실행: {joints}")
```

---

### 5. 그리퍼 제어 (Modbus)

**OnRobot RG2 그리퍼**:
```python
# gripper_controller.py
class GripperController:
    def __init__(self, host='192.168.1.1', port=502):
        self.client = ModbusTcpClient(host=host, port=port)
        self.client.connect()
        
        # 레지스터 주소
        self.TARGET_WIDTH_ADDR = 0x0101
        self.TARGET_FORCE_ADDR = 0x0102
        self.CONTROL_ADDR = 0x0103
        
    def grip(self, width_mm=0, force_N=40):
        """그리퍼 잡기"""
        # 목표 폭 설정 (mm 단위)
        self.client.write_register(self.TARGET_WIDTH_ADDR, int(width_mm * 10))
        
        # 목표 힘 설정 (N 단위)
        self.client.write_register(self.TARGET_FORCE_ADDR, int(force_N * 10))
        
        # 그립 명령 전송
        self.client.write_register(self.CONTROL_ADDR, 0x0001)
        
        # 완료 대기
        time.sleep(1.0)
    
    def release(self):
        """그리퍼 열기"""
        self.client.write_register(self.TARGET_WIDTH_ADDR, 1100)  # 110mm
        self.client.write_register(self.CONTROL_ADDR, 0x0001)
        time.sleep(1.0)
    
    def get_current_width(self):
        """현재 폭 읽기"""
        result = self.client.read_holding_registers(0x0201, 1)
        return result.registers[0] / 10.0  # mm
```

**RViz 동기화**:
```python
# gripper_state_publisher.py
class GripperStatePublisher(Node):
    def __init__(self):
        super().__init__('gripper_state_publisher')
        
        # Joint State 퍼블리셔
        self.joint_pub = self.create_publisher(
            JointState,
            '/gripper/joint_states',
            10
        )
        
        # Modbus 읽기 (10Hz)
        self.create_timer(0.1, self.publish_gripper_state)
    
    def publish_gripper_state(self):
        """그리퍼 상태를 Joint State로 변환"""
        width_mm = self.gripper.get_current_width()
        
        # 110mm → 0.0 rad, 0mm → 0.8 rad (근사)
        finger_angle = (110 - width_mm) / 110 * 0.8
        
        msg = JointState()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = ['rg2_gripper_finger1_joint', 'rg2_gripper_finger2_joint']
        msg.position = [finger_angle, -finger_angle]
        
        self.joint_pub.publish(msg)
```

---

### 6. 충돌 복구 시스템

**자동 충돌 감지**:
```python
# collision_recovery_node.py
class CollisionRecoveryNode(Node):
    def __init__(self):
        super().__init__('collision_recovery')
        
        # 로봇 상태 구독
        self.create_subscription(
            RobotState,
            '/dsr01/state',
            self.state_callback,
            10
        )
        
        self.last_error_code = 0
        
    def state_callback(self, msg):
        """로봇 에러 체크"""
        if msg.robot_state == ROBOT_STATE_EMERGENCY:
            self.get_logger().error("긴급 정지 감지!")
            self.execute_recovery()
        
        if msg.robot_error != 0 and msg.robot_error != self.last_error_code:
            self.get_logger().warn(f"충돌 감지: Error {msg.robot_error}")
            self.execute_recovery()
            self.last_error_code = msg.robot_error
    
    def execute_recovery(self):
        """충돌 복구 시퀀스"""
        self.get_logger().info("복구 시작...")
        
        # 1. 에러 리셋
        DR_ResetError()
        time.sleep(0.5)
        
        # 2. 현재 위치에서 약간 뒤로
        current_pose = DR_GetCurrentPosj()
        recovery_pose = [
            current_pose[0],
            current_pose[1] - 5.0,  # J2 5도 뒤로
            current_pose[2],
            current_pose[3],
            current_pose[4],
            current_pose[5]
        ]
        DR_MoveJ(recovery_pose, vel=10, acc=10)
        time.sleep(2.0)
        
        # 3. 홈 포지션으로 복귀
        DR_MoveJ(HOME_JOINTS, vel=30, acc=30)
        time.sleep(3.0)
        
        self.get_logger().info("복구 완료!")
```

---

## 🛠️ 설치 및 실행

### 환경 요구사항

**하드웨어**:
- Doosan M0609 로봇 팔
- Intel RealSense D435 카메라
- OnRobot RG2 그리퍼
- 제어 PC: Ubuntu 22.04

**소프트웨어**:
- ROS2 Humble
- Python 3.10+
- Node.js 18+
- CUDA 11.8+ (YOLOv8 GPU 가속)

### 설치 가이드

**1. 저장소 클론**:
```bash
git clone https://github.com/Rokey-C1/bbangee.git
cd bbangee
```

**2. Python 가상환경**:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r bbangee/backend/requirements.txt
```

**3. ROS2 워크스페이스 빌드**:
```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash

colcon build --packages-select \
    face_tracking \
    ros2_web_bridge \
    gripper_rviz_sync \
    camera_utils \
    voice_auth \
    sam3_grip_detection

source install/setup.bash
```

**4. Frontend 의존성**:
```bash
cd bbangee/frontend
npm install
```

**5. 환경 변수 설정**:
```bash
# ROS2
export ROS_DOMAIN_ID=64
export ROS_LOCALHOST_ONLY=0

# Backend IP (프론트엔드 .env)
echo "VITE_API_BASE_URL=http://192.168.10.50:8000" > bbangee/frontend/.env
```

---

## 📊 시스템 실행

### 전체 시스템 한번에 실행
```bash
cd bbangee
chmod +x start_all.sh
./start_all.sh
```

**실행 순서**:
1. Doosan 로봇 드라이버 (RViz 포함)
2. 그리퍼 RViz 동기화 (Modbus)
3. RealSense 카메라
4. Image Flip 노드
5. Face Detection 노드 (YOLOv8)
6. Face Tracking 노드 (EKF)
7. Joint Tracking 노드
8. ROS2 Web Bridge 노드들
9. Collision Recovery 노드
10. FastAPI 백엔드 (포트 8000)
11. React 프론트엔드 (포트 5173)

**접속**:
- 🌐 프론트엔드: `http://localhost:5173`
- 📡 백엔드 API: `http://localhost:8000`
- 📚 Swagger 문서: `http://localhost:8000/docs`

---

## 📈 성능 지표

| 항목 | 측정값 |
|------|--------|
| 얼굴 검출 속도 | 30 FPS (640x480) |
| EKF 필터링 지연 | <50ms |
| 로봇 응답 시간 | <200ms |
| IFF 판별 시간 | 3~5초 (신뢰도 확보) |
| WebSocket 지연 | <100ms |
| 카메라 스트리밍 | 15 FPS (MJPEG) |

---

## 🎓 기술적 도전과 해결

### 1. 실시간 얼굴 추적의 지터 문제
**문제**: YOLOv8 검출 결과의 급격한 변동으로 로봇이 떨림

**해결**:
- EKF (Extended Kalman Filter) 도입
- 프로세스 노이즈 0.1, 측정 노이즈 5.0으로 튜닝
- PID 제어기 적용 (Kp=0.015, Ki=0.001, Kd=0.005)

### 2. IFF 피아식별 신뢰도 향상
**문제**: 단일 프레임 CNN 예측 결과가 불안정

**해결**:
- 계층적 베이지안 필터 설계 (2단계)
- 시간 감쇠 + 학습률 조정으로 노이즈 제거
- 확정 임계값 설정 (Stage 1: 75%, Stage 2: 70%)

### 3. 웹-ROS2 통신 동기화
**문제**: WebSocket과 ROS2 토픽 간 타이밍 불일치

**해결**:
- ROS2 Web Bridge 노드 분리
- 명령 파일 기반 비동기 통신 (JSON)
- WebSocket으로 실시간 상태 스트리밍

### 4. 그리퍼 RViz 시각화
**문제**: Modbus 제어 그리퍼가 RViz에 표시 안 됨

**해결**:
- Joint State Publisher 커스텀 노드 작성
- Modbus 읽기 → Joint State 변환 (10Hz)
- Joint State Merger로 로봇 + 그리퍼 병합

---

## 🔗 관련 링크

- **GitHub Repository**: [Rokey-C1/bbangee](https://github.com/Rokey-C1/bbangee)
- **Doosan Robotics**: [doosanrobotics.com](https://www.doosanrobotics.com/)
- **RealSense SDK**: [github.com/IntelRealSense](https://github.com/IntelRealSense/realsense-ros)
- **YOLOv8 Face**: [github.com/akanametov/yolov8-face](https://github.com/akanametov/yolov8-face)
- **OnRobot RG2**: [onrobot.com/en/products/rg2-gripper](https://onrobot.com/en/products/rg2-gripper)

---

## 📝 라이선스

This project uses:
- Doosan Robotics SDK (Proprietary)
- ROS2 Humble (Apache 2.0)
- FastAPI (MIT License)
- React (MIT License)
- YOLOv8 (AGPL-3.0)

---

**Last Updated**: 2025-12-21  
**Contact**: [GitHub @Rokey-C1](https://github.com/Rokey-C1)
