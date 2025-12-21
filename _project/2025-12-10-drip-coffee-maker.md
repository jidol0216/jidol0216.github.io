---
title: "Drip Coffee Maker - Full-Stack Robotic Barista System"
short_title: "Robotic Barista"
date: 2025-12-10
collection: project
permalink: /projects/drip-coffee-maker/
excerpt: "Doosan M0609 로봇 팔을 활용한 자동 드립 커피 제조 시스템. React 프론트엔드, FastAPI 백엔드, ROS2 로봇 제어를 통합한 풀스택 바리스타 솔루션"
---

# ☕ Drip Coffee Maker - Robotic Barista System

> **프로젝트 이름**: Drip Coffee Maker  
> **기간**: 2025  
> **플랫폼**: ROS2 Humble · Doosan M0609 · FastAPI · React  
> **GitHub**: [Rokey-C1/drip_coffee_maker](https://github.com/Rokey-C1/drip_coffee_maker)

---

## 🎯 프로젝트 개요

**Drip Coffee Maker**는 Doosan 로봇 팔(M0609)을 이용한 완전 자동화된 드립 커피 제조 시스템입니다. 바리스타가 웹 UI에서 레시피를 입력하면, 백엔드가 이를 관리하고 ROS2를 통해 실제 로봇이 드립 포트를 제어하여 커피를 추출합니다.

### 핵심 기능
- 🌐 **웹 기반 UI**: iPad/브라우저에서 커피 레시피 관리
- 🤖 **로봇 자동 제어**: Doosan M0609 암을 이용한 정밀한 드립 포트 제어
- 📊 **레시피 관리**: SQLite 기반 원두/레시피 데이터베이스
- 🔄 **실시간 모니터링**: 추출 진행 상황 실시간 표시
- ⚖️ **로드셀 통합**: 물 무게 측정 및 피드백

---

## 🏗️ 시스템 아키텍처

### 전체 구조
```
┌─────────────────────────────────────────────────────────────┐
│                      Full-Stack System                      │
├──────────────┬──────────────────┬──────────────────────────┤
│  Frontend    │     Backend      │         ROS2             │
│  (React)     │    (FastAPI)     │  (Robot Control)         │
├──────────────┼──────────────────┼──────────────────────────┤
│ • Vite       │ • Python 3.10    │ • Doosan M0609          │
│ • TypeScript │ • SQLite DB      │ • Gripper Control       │
│ • UI/UX      │ • REST API       │ • Motion Planning       │
│ • Recipe     │ • ROS Bridge     │ • Load Cell Sensor      │
│   Management │ • CORS           │ • Recipe Execution      │
└──────────────┴──────────────────┴──────────────────────────┘
         │              │                    │
         └──────────────┴────────────────────┘
                        │
                  Network (REST API)
```

### 디렉토리 구조
```
drip_coffee_maker/
├── frontend/                  # React + Vite + TypeScript
│   ├── src/
│   │   ├── screens/          # UI 화면들
│   │   ├── api.ts            # Backend API 호출
│   │   ├── types.ts          # TypeScript 타입 정의
│   │   └── App.tsx           # 메인 앱
│   └── package.json
│
├── backend/                   # FastAPI Python Server
│   ├── app/
│   │   ├── main.py           # FastAPI 라우터
│   │   ├── models.py         # SQLAlchemy 모델
│   │   ├── schemas.py        # Pydantic 스키마
│   │   ├── database.py       # DB 연결
│   │   └── ros_bridge.py     # ROS2 통신 브리지
│   ├── create_db.py          # DB 초기화
│   └── requirements.txt
│
└── ros2_ws/                   # ROS2 Workspace
    └── src/
        ├── baro_bringup/     # 🔧 통합 실행 패키지
        │   ├── recipe_manager.py      # 레시피 실행 매니저
        │   └── loadcell_node.py       # 무게 센서 노드
        ├── baro_utils/       # 🔧 유틸리티 노드
        │   ├── go_home.py            # 홈 포지션 이동
        │   └── gripper_node.py       # 그리퍼 제어
        └── motion_controller/ # 🔧 모션 제어
            ├── pouring_node.py       # 드립 모션
            ├── posing_node.py        # 자세 제어
            └── simple_pouring.py     # 단순 테스트
```

---

## 🚀 주요 기능

### 1. Frontend - 레시피 관리 UI

**기술 스택**:
- React 18 + Vite
- TypeScript
- Responsive Design (iPad 최적화)

**주요 화면**:
1. **원두 관리 (Bean Management)**
   - 원두 정보 등록 (이름, 원산지, 로스팅 레벨)
   - 원두 목록 조회 및 선택

2. **레시피 관리 (Recipe Management)**
   - 레시피 생성 및 편집
   - 추출 단계별 파라미터 설정:
     - 물 양 (ml)
     - 기울기 속도 (slow/medium/fast)
     - 서클링 크기 (small/medium/large)
     - 서클링 속도 (slow/medium/fast)
   - 맛 노트 및 설명 작성

3. **추출 화면 (Brewing Screen)**
   - 레시피 선택 및 시작
   - 실시간 진행 상태 표시
   - 현재 단계 모니터링

**API 통신**:
```typescript
// api.ts
export const startBrewing = async (recipeId: number) => {
  const response = await fetch(`${API_BASE}/brew/start`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ recipe_id: recipeId })
  });
  return response.json();
};

export const getBrewStatus = async () => {
  const response = await fetch(`${API_BASE}/brew/status`);
  return response.json();
};
```

---

### 2. Backend - API Server

**기술 스택**:
- FastAPI (Python 3.10)
- SQLAlchemy (ORM)
- SQLite (Database)
- Uvicorn (ASGI Server)

**데이터베이스 모델**:
```python
# models.py
class Bean(Base):
    __tablename__ = "beans"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    origin = Column(String)
    roast_level = Column(String)
    flavor_notes = Column(String)

class Recipe(Base):
    __tablename__ = "recipes"
    id = Column(Integer, primary_key=True)
    bean_id = Column(Integer, ForeignKey("beans.id"))
    name = Column(String, nullable=False)
    subtitle = Column(String)
    description = Column(Text)
    process = Column(String)
    flavor_tip = Column(Text)
    image_key = Column(String)
    steps_json = Column(Text)  # JSON serialized PourStep[]
```

**주요 API 엔드포인트**:
```python
# main.py
@app.get("/beans")
def get_beans(db: Session = Depends(get_db)):
    """원두 목록 조회"""
    return db.query(models.Bean).all()

@app.post("/recipes")
def create_recipe(recipe: RecipeCreate, db: Session = Depends(get_db)):
    """레시피 생성"""
    steps_json = json.dumps([s.dict() for s in recipe.steps])
    db_recipe = models.Recipe(
        bean_id=recipe.bean_id,
        name=recipe.name,
        steps_json=steps_json
    )
    db.add(db_recipe)
    db.commit()
    return db_recipe

@app.post("/brew/start")
def start_brewing(request: BrewRequest):
    """추출 시작 - ROS2에 레시피 전달"""
    recipe = db.query(Recipe).filter_by(id=request.recipe_id).first()
    ros_bridge.send_recipe_to_ros(recipe)
    return {"status": "started"}

@app.get("/brew/status")
def get_brew_status():
    """현재 추출 상태 조회"""
    return ros_bridge.get_current_status()
```

**ROS Bridge**:
```python
# ros_bridge.py
import rclpy
from std_msgs.msg import String

class RosBridge:
    def __init__(self):
        rclpy.init()
        self.node = rclpy.create_node('ros_bridge')
        self.recipe_pub = self.node.create_publisher(String, '/recipe_command', 10)
    
    def send_recipe_to_ros(self, recipe: Recipe):
        """레시피를 ROS2로 전송"""
        steps_data = json.loads(recipe.steps_json)
        message = String()
        message.data = json.dumps({
            'recipe_id': recipe.id,
            'steps': steps_data
        })
        self.recipe_pub.publish(message)
```

---

### 3. ROS2 - Robot Control

**기술 스택**:
- ROS2 Humble
- Doosan Robotics SDK
- Python 3.10
- Custom Nodes

#### 3.1 Recipe Manager (레시피 실행 매니저)

**핵심 로직**:
```python
# recipe_manager.py
class RecipeManager(Node):
    def __init__(self):
        super().__init__('recipe_manager')
        
        # 구독자: 백엔드로부터 레시피 수신
        self.create_subscription(
            String, 
            '/recipe_command', 
            self.recipe_callback, 
            10
        )
        
        # 퍼블리셔: 상태 업데이트
        self.status_pub = self.create_publisher(
            String, 
            '/brew_status', 
            10
        )
        
    def recipe_callback(self, msg):
        """레시피 수신 및 실행"""
        recipe_data = json.loads(msg.data)
        steps = recipe_data['steps']
        
        self.execute_recipe(steps)
    
    def execute_recipe(self, steps: List[PourStep]):
        """단계별 추출 실행"""
        for i, step in enumerate(steps):
            self.get_logger().info(f"Step {i+1}/{len(steps)}: {step}")
            
            # 1. 물 양에 따른 기울기 각도 계산
            tilt_angle = self.calculate_tilt_angle(step)
            
            # 2. 서클링 파라미터 설정
            circle_radius = CIRCLE_SIZE_TO_RADIUS_MM[step.circle_size]
            circle_speed_mode = step.circle_speed
            
            # 3. 로봇 모션 실행
            self.execute_pour_motion(
                tilt_angle=tilt_angle,
                circle_radius=circle_radius,
                circle_speed_mode=circle_speed_mode,
                duration=step.water / 10 * SPEED_PROFILE[step.speed]['time_per_10ml']
            )
            
            # 4. 상태 업데이트
            self.publish_status(f"Step {i+1} completed")
            
            # 5. 다음 단계 전 대기
            time.sleep(2.0)
    
    def calculate_tilt_angle(self, step: PourStep) -> float:
        """속도에 따른 기울기 각도 계산"""
        profile = SPEED_PROFILE[step.speed]
        base_angle = BASE_TILT_START_DEG
        delta_angle = profile['delta_deg']
        
        return min(base_angle + delta_angle, MAX_TILT_END_DEG)
```

**드립 모션 프로파일**:
| 속도 | 기울기 각도 | 10ml당 시간 | 서클링 반경 |
|------|------------|-------------|------------|
| slow | 15° + 9° = 24° | 2.8초 | small: 8mm |
| medium | 15° + 10° = 25° | 2.0초 | medium: 12mm |
| fast | 15° + 13° = 28° | 1.5초 | large: 16mm |

---

#### 3.2 Motion Controller (모션 제어)

**Pouring Node**:
```python
# pouring_node.py
def pour_over(
    tilt_angle: float,
    circle_radius_mm: float,
    circle_speed_mode: str,
    duration_sec: float
):
    """드립 포트 기울이기 + 서클링"""
    
    # 1. 초기 위치로 이동
    DR_MoveJ(HOME_POSITION, vel=60, acc=60)
    
    # 2. 드립 시작 위치로 이동 (드리퍼 상단)
    DR_MoveL(DRIP_START_POSITION, vel=100, acc=100)
    
    # 3. 기울이기 (물 흐르기 시작)
    tilt_target = calculate_tilt_pose(tilt_angle)
    DR_MoveL(tilt_target, vel=30, acc=30)
    
    # 4. 서클링 (나선형 물줄기)
    num_circles = int(duration_sec / CIRCLE_PERIOD[circle_speed_mode])
    for i in range(num_circles):
        circle_path = generate_circle_trajectory(
            center=tilt_target,
            radius_mm=circle_radius_mm,
            num_points=20
        )
        DR_MoveL(circle_path, vel=50, acc=50)
    
    # 5. 기울기 복원 (물 멈춤)
    DR_MoveL(DRIP_START_POSITION, vel=30, acc=30)
    
    # 6. 홈 위치로 복귀
    DR_MoveJ(HOME_POSITION, vel=60, acc=60)
```

**3D 궤적 생성**:
```python
def generate_circle_trajectory(center, radius_mm, num_points):
    """서클링을 위한 원형 궤적 생성"""
    trajectory = []
    for i in range(num_points):
        angle = 2 * math.pi * i / num_points
        x = center[0] + radius_mm * math.cos(angle)
        y = center[1] + radius_mm * math.sin(angle)
        z = center[2]
        rx, ry, rz = center[3], center[4], center[5]
        
        trajectory.append([x, y, z, rx, ry, rz])
    
    return trajectory
```

---

#### 3.3 Utility Nodes

**Gripper Control**:
```python
# gripper_node.py
def gripper_control(open: bool):
    """그리퍼 열기/닫기"""
    if open:
        DR_SetDigitalOutput(1, True)   # Open signal
        DR_SetDigitalOutput(2, False)
    else:
        DR_SetDigitalOutput(1, False)
        DR_SetDigitalOutput(2, True)   # Close signal
```

**Load Cell Node**:
```python
# loadcell_node.py
class LoadCellNode(Node):
    def __init__(self):
        super().__init__('loadcell_node')
        
        # 무게 센서 퍼블리셔
        self.weight_pub = self.create_publisher(Float32, '/water_weight', 10)
        
        # 시리얼 통신 (아두이노 로드셀)
        self.serial = serial.Serial('/dev/ttyACM0', 9600)
        
        self.create_timer(0.1, self.read_weight)
    
    def read_weight(self):
        """무게 읽기 및 퍼블리시"""
        if self.serial.in_waiting:
            weight_str = self.serial.readline().decode('utf-8').strip()
            weight = float(weight_str)
            
            msg = Float32()
            msg.data = weight
            self.weight_pub.publish(msg)
```

---

## 🛠️ 설치 및 실행

### 환경 요구사항

**하드웨어**:
- Doosan M0609 로봇 팔
- OnRobot RG2 그리퍼 (옵션)
- 로드셀 (HX711 + 아두이노)
- 제어 PC: Ubuntu 22.04

**소프트웨어**:
- ROS2 Humble
- Python 3.10+
- Node.js 18+
- SQLite 3

### 설치 가이드

**1. 저장소 클론**:
```bash
git clone https://github.com/Rokey-C1/drip_coffee_maker.git
cd drip_coffee_maker
```

**2. Backend 설정**:
```bash
cd backend
pip install -r requirements.txt
python create_db.py  # DB 초기화
uvicorn app.main:app --reload
```

**3. Frontend 설정**:
```bash
cd frontend
npm install
npm run dev          # 로컬 개발 서버
npm run host         # 네트워크 호스팅 (iPad 접속용)
```

**4. ROS2 설정**:
```bash
cd ros2_ws
colcon build
source install/setup.bash

# 로봇 연결 (가상 모드)
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual host:=127.0.0.1 port:=12345 model:=m0609

# 레시피 매니저 실행
ros2 run baro_bringup recipe_manager
```

**5. 통합 테스트**:
```bash
# 홈 포지션 이동
ros2 run baro_utils go_home

# 그리퍼 테스트
ros2 run baro_utils gripper_node

# 단순 드립 테스트
ros2 run motion_controller simple_pouring
```

---

## 📊 시스템 플로우

### 추출 프로세스
```
1. [Frontend] 바리스타가 레시피 선택 및 시작 버튼 클릭
         ↓
2. [Backend] POST /brew/start 수신
         ↓
3. [Backend] DB에서 레시피 조회 (steps_json)
         ↓
4. [ROS Bridge] ROS2 토픽 /recipe_command로 전송
         ↓
5. [ROS2 Recipe Manager] 레시피 수신 및 파싱
         ↓
6. [ROS2] 단계별 실행 루프:
    - Step 1: 물 20ml, slow, small circle
      → 기울기 24°, 서클링 8mm, 5.6초
    - Step 2: 물 30ml, medium, medium circle
      → 기울기 25°, 서클링 12mm, 6.0초
    - Step 3: 물 50ml, fast, large circle
      → 기울기 28°, 서클링 16mm, 7.5초
         ↓
7. [ROS2] 각 단계마다 /brew_status 토픽으로 진행 상황 전송
         ↓
8. [Backend] GET /brew/status로 상태 조회 가능
         ↓
9. [Frontend] 실시간 UI 업데이트 (폴링 or WebSocket)
         ↓
10. [완료] 모든 단계 종료 후 홈 포지션 복귀
```

---

## 🎓 기술적 도전과 해결

### 1. 정밀한 물 제어
**문제**: 로봇 팔의 기울기와 실제 물 양의 불일치

**해결**:
- 로드셀을 이용한 실시간 무게 측정
- 속도별 기울기 프로파일 실험 및 캘리브레이션
- 10ml당 시간 기반 제어 (empirical data)

### 2. Frontend-Backend-ROS2 통합
**문제**: 3개의 독립적인 시스템 간 동기화

**해결**:
- REST API 기반 명확한 인터페이스 정의
- ROS Bridge 모듈로 백엔드-ROS2 분리
- CORS 설정으로 네트워크 접근 허용

### 3. 서클링 모션의 부드러움
**문제**: 급격한 방향 전환으로 인한 물 튐

**해결**:
- 20개 포인트로 원형 궤적 세분화
- 속도/가속도 제한 (vel=50, acc=50)
- 서클링 속도 모드별 주기 조정

---

## 📈 성능 지표

| 항목 | 측정값 |
|------|--------|
| 추출 정확도 | ±5ml (100ml 기준) |
| 레시피 실행 시간 | 평균 2~5분 |
| API 응답 시간 | <100ms |
| UI 렌더링 | 60 FPS (Vite 최적화) |
| 로봇 위치 정밀도 | ±0.1mm (Doosan 사양) |

---

## 🔗 관련 링크

- **GitHub Repository**: [Rokey-C1/drip_coffee_maker](https://github.com/Rokey-C1/drip_coffee_maker)
- **Doosan Robotics**: [doosanrobotics.com](https://www.doosanrobotics.com/)
- **FastAPI 문서**: [fastapi.tiangolo.com](https://fastapi.tiangolo.com/)
- **React 공식 문서**: [react.dev](https://react.dev/)

---

## 📝 라이선스

This project uses:
- Doosan Robotics SDK (Proprietary)
- FastAPI (MIT License)
- React (MIT License)
- ROS2 Humble (Apache 2.0)

---

**Last Updated**: 2025-12-21  
**Contact**: [GitHub @Rokey-C1](https://github.com/Rokey-C1)
