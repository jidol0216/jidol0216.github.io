---
title: "두산 로봇 Docker 사용 가이드"
collection: study
type: "Guide"
permalink: /study/doosan-docker-usage
date: 2025-12-21
excerpt: "Doosan Robot Emulator Docker 환경 구축 및 실행 가이드. Virtual Mode 테스트와 RViz2 연동까지."
---

# 두산 로봇 도커 사용법 가이드

## 목차
1. [개요](#개요)
2. [도커 에뮬레이터 설치](#도커-에뮬레이터-설치)
3. [도커 컨테이너 관리](#도커-컨테이너-관리)
4. [로봇 실행 방법](#로봇-실행-방법)
5. [문제 해결](#문제-해결)
6. [고급 사용법](#고급-사용법)

---

## 개요

두산 로봇은 **Virtual Mode**에서 작동할 때 도커 컨테이너 기반 에뮬레이터를 사용합니다.

### 왜 도커를 사용하나요?

- ✅ **실제 로봇 없이 테스트 가능** - 가상 환경에서 로봇 동작 시뮬레이션
- ✅ **일관된 환경** - 어떤 시스템에서도 동일하게 작동
- ✅ **자동 관리** - ROS2 launch 파일이 자동으로 시작/종료
- ✅ **격리된 실행** - 시스템에 영향 없이 안전하게 실행

### 에뮬레이터 정보

- **이미지 이름**: `doosanrobot/dsr_emulator:3.0.1`
- **컨테이너 이름**: `dsr01_emulator` (기본값)
- **포트**: `12345` (기본값)
- **호스트**: `127.0.0.1` (localhost)

---

## 도커 에뮬레이터 설치

### 1. 자동 설치 (권장)

```bash
cd /home/jidol/ros2_ws/src/doosan-robot2
sudo ./install_emulator.sh
```

**출력 예시:**
```
3.0.1: Pulling from doosanrobot/dsr_emulator
✓ Downloaded successfully
```

### 2. 수동 설치

```bash
docker pull doosanrobot/dsr_emulator:3.0.1
```

### 3. 설치 확인

```bash
docker images | grep dsr_emulator
```

**예상 결과:**
```
doosanrobot/dsr_emulator   3.0.1   abc123def456   2 weeks ago   1.5GB
```

---

## 도커 컨테이너 관리

### 📌 기본 명령어

#### 1. 실행 중인 컨테이너 확인

```bash
docker ps
```

**로봇 실행 중일 때:**
```
CONTAINER ID   IMAGE                              COMMAND       STATUS
abc123def456   doosanrobot/dsr_emulator:3.0.1    "./drcf"      Up 2 minutes
```

#### 2. 모든 컨테이너 확인 (중지된 것 포함)

```bash
docker ps -a
```

#### 3. 두산 컨테이너만 확인

```bash
docker ps -a | grep dsr
```

#### 4. 컨테이너 로그 확인

```bash
# 실시간 로그
docker logs -f dsr01_emulator

# 최근 50줄만 보기
docker logs --tail 50 dsr01_emulator
```

#### 5. 컨테이너 수동 중지

```bash
docker stop dsr01_emulator
```

#### 6. 컨테이너 삭제

```bash
# 중지 후 삭제
docker stop dsr01_emulator
docker rm dsr01_emulator

# 강제 삭제 (실행 중이어도)
docker rm -f dsr01_emulator
```

#### 7. 모든 중지된 컨테이너 삭제

```bash
docker container prune
```

#### 8. 에뮬레이터 이미지 삭제 (재설치 시)

```bash
docker rmi doosanrobot/dsr_emulator:3.0.1
```

---

## 로봇 실행 방법

### 🚀 방법 1: ROS2 Launch 사용 (권장)

ROS2 launch 파일을 사용하면 도커가 **자동으로 시작/종료**됩니다.

```bash
# ROS2 환경 설정
source /home/jidol/ros2_ws/install/setup.bash

# Virtual Mode로 로봇 실행
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual \
  host:=127.0.0.1 \
  port:=12345 \
  model:=m1013
```

**실행 단계:**
1. Docker 컨테이너 자동 시작 (`dsr01_emulator`)
2. ROS2 컨트롤러 초기화
3. RViz2 창 열림

**종료 방법:**
- `Ctrl+C` 누르면 도커도 자동 종료됨

### 🚀 방법 2: 도커 수동 실행

필요한 경우 도커를 직접 실행할 수 있습니다.

```bash
docker run -d \
  --name dsr01_emulator \
  -p 12345:12345 \
  doosanrobot/dsr_emulator:3.0.1
```

**옵션 설명:**
- `-d`: 백그라운드 실행 (detached mode)
- `--name`: 컨테이너 이름 지정
- `-p 12345:12345`: 포트 매핑 (호스트:컨테이너)

**확인:**
```bash
docker ps
```

**종료:**
```bash
docker stop dsr01_emulator
docker rm dsr01_emulator
```

### 🎯 로봇 모델 선택

```bash
# M1013 모델 (기본)
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual model:=m1013

# M0617 모델
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual model:=m0617

# M0609 모델
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual model:=m0609
```

**지원 모델:**
- `m0609`, `m0617`, `m1013`, `m1509`
- `a0509`, `a0912`, `h2017`, `h2515`

---

## 문제 해결

### ❌ 문제 1: "포트가 이미 사용 중입니다"

**증상:**
```
Error: Port 12345 is already in use
```

**원인:** 이전 컨테이너가 아직 실행 중

**해결방법:**
```bash
# 1. 실행 중인 컨테이너 확인
docker ps -a | grep dsr

# 2. 강제 종료 및 삭제
docker rm -f dsr01_emulator

# 3. 다시 실행
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual
```

### ❌ 문제 2: "도커 이미지를 찾을 수 없습니다"

**증상:**
```
Unable to find image 'doosanrobot/dsr_emulator:3.0.1' locally
```

**해결방법:**
```bash
# 에뮬레이터 재설치
cd /home/jidol/ros2_ws/src/doosan-robot2
sudo ./install_emulator.sh
```

### ❌ 문제 3: "연결할 수 없습니다"

**증상:**
```
Failed to connect to 127.0.0.1:12345
```

**해결방법:**
```bash
# 1. 도커가 실행 중인지 확인
docker ps

# 2. 컨테이너 로그 확인
docker logs dsr01_emulator

# 3. 도커 재시작
docker restart dsr01_emulator

# 4. 안 되면 완전히 재시작
docker rm -f dsr01_emulator
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual
```

### ❌ 문제 4: "도커 권한 오류"

**증상:**
```
permission denied while trying to connect to the Docker daemon
```

**해결방법:**
```bash
# 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER

# 재로그인 필요 (또는 재부팅)
newgrp docker

# 확인
docker ps
```

### ❌ 문제 5: 컨테이너가 계속 종료됨

**증상:**
```
docker ps -a
STATUS: Exited (1) 5 seconds ago
```

**해결방법:**
```bash
# 1. 로그에서 오류 확인
docker logs dsr01_emulator

# 2. 모든 컨테이너 정리
docker rm -f $(docker ps -aq)

# 3. 이미지 재다운로드
docker rmi doosanrobot/dsr_emulator:3.0.1
cd /home/jidol/ros2_ws/src/doosan-robot2
sudo ./install_emulator.sh
```

---

## 고급 사용법

### 📊 도커 리소스 모니터링

```bash
# CPU, 메모리 사용량 실시간 확인
docker stats dsr01_emulator

# 디스크 사용량
docker system df
```

### 🔧 컨테이너 내부 접근

```bash
# 컨테이너 셸 접속
docker exec -it dsr01_emulator /bin/bash

# 특정 명령 실행
docker exec dsr01_emulator ls -la /
```

### 🧹 도커 전체 정리

```bash
# 중지된 컨테이너 모두 삭제
docker container prune -f

# 사용하지 않는 이미지 삭제
docker image prune -a -f

# 전체 정리 (컨테이너, 이미지, 볼륨, 네트워크)
docker system prune -a -f
```

**⚠️ 주의:** `prune -a` 명령은 도커의 모든 이미지를 삭제합니다!

### 📝 도커 네트워크 확인

```bash
# 네트워크 목록
docker network ls

# 컨테이너의 네트워크 상세 정보
docker inspect dsr01_emulator | grep -A 20 "NetworkSettings"
```

### 🎛️ 커스텀 포트 사용

```bash
# 다른 포트로 실행 (예: 15000)
docker run -d \
  --name dsr01_emulator_custom \
  -p 15000:12345 \
  doosanrobot/dsr_emulator:3.0.1

# ROS2에서 연결
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual \
  host:=127.0.0.1 \
  port:=15000 \
  model:=m1013
```

### 🔄 여러 로봇 동시 실행

```bash
# 첫 번째 로봇
docker run -d \
  --name dsr01_emulator \
  -p 12345:12345 \
  doosanrobot/dsr_emulator:3.0.1

# 두 번째 로봇
docker run -d \
  --name dsr02_emulator \
  -p 12346:12345 \
  doosanrobot/dsr_emulator:3.0.1
```

```bash
# 첫 번째 로봇 연결
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual host:=127.0.0.1 port:=12345 model:=m1013 name:=dsr01

# 두 번째 로봇 연결 (다른 터미널)
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
  mode:=virtual host:=127.0.0.1 port:=12346 model:=m1013 name:=dsr02
```

---

## 빠른 참조

### 자주 사용하는 명령어

```bash
# 로봇 실행
source /home/jidol/ros2_ws/install/setup.bash
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual

# 도커 상태 확인
docker ps

# 로그 보기
docker logs -f dsr01_emulator

# 컨테이너 종료
docker stop dsr01_emulator && docker rm dsr01_emulator

# 전체 재시작
docker rm -f dsr01_emulator
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual
```

### 유용한 별칭 (Alias) 설정

`~/.bashrc`에 추가하면 편리합니다:

```bash
# 두산 로봇 관련 별칭
alias dsr-start='source /home/jidol/ros2_ws/install/setup.bash && ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual host:=127.0.0.1 port:=12345 model:=m1013'
alias dsr-stop='docker stop dsr01_emulator && docker rm dsr01_emulator'
alias dsr-status='docker ps -a | grep dsr'
alias dsr-logs='docker logs -f dsr01_emulator'
alias dsr-clean='docker rm -f $(docker ps -aq -f name=dsr)'
```

**사용법:**
```bash
# 별칭 적용
source ~/.bashrc

# 로봇 실행
dsr-start

# 상태 확인
dsr-status

# 로그 보기
dsr-logs

# 종료
dsr-stop
```

---

## 요약

### Virtual Mode 실행 순서

1. **환경 설정**
   ```bash
   source /home/jidol/ros2_ws/install/setup.bash
   ```

2. **로봇 실행** (도커 자동 시작)
   ```bash
   ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual
   ```

3. **RViz2에서 확인**
   - 로봇 3D 모델 표시
   - 관절 상태 실시간 업데이트

4. **종료** (도커 자동 종료)
   ```bash
   Ctrl+C
   ```

### 핵심 포인트

- ✅ **자동 관리**: ROS2 launch가 도커를 자동으로 시작/종료
- ✅ **간단한 사용**: `mode:=virtual` 하나면 모든 설정 완료
- ✅ **문제 해결**: 대부분 `docker rm -f dsr01_emulator`로 해결
- ✅ **로그 확인**: `docker logs dsr01_emulator`로 디버깅

---

## 참고 자료

- [두산 로봇 공식 문서](https://github.com/doosan-robotics/doosan-robot2)
- [Docker 공식 문서](https://docs.docker.com/)
- [ROS2 Humble 문서](https://docs.ros.org/en/humble/)

**관련 가이드:**
- `DOOSAN_ROS2_GUIDE.md` - 두산 ROS2 패키지 전체 가이드
- `DOCKER_GUIDE.md` - Docker 초보자 가이드
- `GPU_SETUP_GUIDE.md` - GPU 설정 가이드

---

*최종 업데이트: 2025년 12월 21일*
