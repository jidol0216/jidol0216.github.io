---
title: "Docker 초보자 완벽 가이드"
collection: ros2
type: "Guide"
permalink: /ros2/docker-guide
date: 2025-12-21
excerpt: "Docker 입문부터 실전까지. 컨테이너 개념, 설치, 주요 명령어, 이미지 빌드, Docker Compose 활용법."
---

# 도커(Docker) 완전 초보 가이드

> 두산 로봇 Virtual Mode를 위한 도커 설명서  
> 작성일: 2025년 12월 21일

---

## 📋 목차

1. [도커란 무엇인가?](#1-도커란-무엇인가)
2. [왜 도커를 사용해야 하나?](#2-왜-도커를-사용해야-하나)
3. [도커 설치하기](#3-도커-설치하기)
4. [도커 기본 개념](#4-도커-기본-개념)
5. [도커 기본 명령어](#5-도커-기본-명령어)
6. [두산 로봇에서 도커 사용하기](#6-두산-로봇에서-도커-사용하기)
7. [자주 묻는 질문](#7-자주-묻는-질문)

---

## 1. 도커란 무엇인가?

### 1.1 쉽게 설명하면

**도커 = 가상의 컴퓨터를 만드는 도구**

컴퓨터 안에 또 다른 작은 컴퓨터를 만들어서, 그 안에서 프로그램을 실행하는 기술입니다.

### 1.2 비유로 이해하기

```
🏠 실제 컴퓨터 (Ubuntu 22.04)
  └─ 📦 도커 컨테이너 (가상 컴퓨터)
      └─ 🤖 두산 로봇 에뮬레이터 실행
```

**레고 블록처럼:**
- 📦 **컨테이너**: 완성된 레고 세트 (프로그램이 돌아가는 환경)
- 🖼️ **이미지**: 레고 설명서 (컨테이너를 만드는 설계도)
- 🏗️ **도커**: 레고를 조립하는 도구

### 1.3 실제 예시

```bash
# 도커 없이 (복잡함)
1. Ubuntu 설치
2. Python 3.8 설치
3. 라이브러리 1000개 설치
4. 환경 변수 설정
5. 프로그램 실행
❌ 문제: 다른 PC에서 또 처음부터 반복

# 도커 사용 (간단함)
docker run doosan-emulator
✅ 한 줄로 모든 환경 자동 설치 및 실행!
```

---

## 2. 왜 도커를 사용해야 하나?

### 2.1 두산 로봇에서 도커를 쓰는 이유

#### 🎯 **실제 로봇 없이 테스트하기 위해서!**

```
실제 로봇 (Real Mode)
- 로봇 필요 💰 (수천만원)
- 공간 필요 🏭
- 위험성 있음 ⚠️

가상 로봇 (Virtual Mode with Docker)
- 로봇 불필요 ✅
- PC만 있으면 됨 💻
- 안전하게 테스트 ✅
```

### 2.2 도커의 장점

#### 1️⃣ **환경 격리**
```bash
# 내 PC 환경
- Python 3.10
- ROS2 Humble

# 도커 컨테이너 안 (독립적)
- Python 2.7
- 두산 로봇 전용 라이브러리
- 충돌 없음!
```

#### 2️⃣ **이식성**
```
PC A에서 작동 → 도커 이미지 → PC B에서도 똑같이 작동
```

#### 3️⃣ **깔끔한 삭제**
```bash
# 일반 설치: 프로그램 삭제 후에도 잔여 파일 남음
# 도커: 컨테이너만 삭제하면 완전히 제거
docker rm my-container  # 끝!
```

### 2.3 두산 로봇 Virtual Mode 동작 원리

```
┌─────────────────────────────────────────┐
│  내 PC (Ubuntu 22.04)                   │
│                                         │
│  ┌───────────────────────────────┐     │
│  │ ROS2 프로그램 (내가 작성한 코드) │     │
│  │  - Python 코드                │     │
│  │  - movej(), movel() 호출       │     │
│  └──────────┬────────────────────┘     │
│             │ 127.0.0.1:12345          │
│             ▼                           │
│  ┌───────────────────────────────┐     │
│  │ 🐳 도커 컨테이너               │     │
│  │  (두산 로봇 에뮬레이터)         │     │
│  │  - 가상 로봇 컨트롤러           │     │
│  │  - 로봇 시뮬레이터              │     │
│  └───────────────────────────────┘     │
└─────────────────────────────────────────┘
```

**쉽게 말하면:**
- 도커 = 가짜 로봇 컨트롤러가 돌아가는 가상 공간
- 내 코드는 이 가짜 컨트롤러와 통신해서 테스트

---

## 3. 도커 설치하기

### 3.1 자동 설치 (추천)

```bash
# 1. 설치 스크립트 다운로드
curl -fsSL https://get.docker.com -o get-docker.sh

# 2. 실행
sudo sh get-docker.sh

# 3. 현재 사용자를 docker 그룹에 추가 (sudo 없이 사용하기 위해)
sudo usermod -aG docker $USER

# 4. 재로그인 (또는 재부팅)
# 로그아웃 후 다시 로그인하거나
newgrp docker
```

### 3.2 설치 확인

```bash
# 도커 버전 확인
docker --version
# 출력 예: Docker version 24.0.7, build afdd53b

# 도커 실행 확인
docker run hello-world
```

**성공 시 메시지:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 3.3 도커 서비스 시작

```bash
# 도커 서비스 시작
sudo systemctl start docker

# 부팅 시 자동 시작 설정
sudo systemctl enable docker

# 상태 확인
sudo systemctl status docker
```

---

## 4. 도커 기본 개념

### 4.1 핵심 용어

#### 🖼️ **이미지 (Image)**
- **정의**: 컨테이너를 만들기 위한 설계도
- **비유**: 앱 설치 파일 (.exe, .apk)
- **예시**: `ubuntu:22.04`, `doosan-emulator:latest`

```bash
# 이미지 = 읽기 전용 템플릿
docker images  # 내 PC에 있는 이미지 목록 보기
```

#### 📦 **컨테이너 (Container)**
- **정의**: 이미지를 실행한 것 (실제로 동작하는 프로그램)
- **비유**: 실행 중인 앱
- **예시**: 도커로 실행된 두산 로봇 에뮬레이터

```bash
# 컨테이너 = 실행 중인 격리된 환경
docker ps  # 실행 중인 컨테이너 목록 보기
```

#### 🐳 **도커 허브 (Docker Hub)**
- **정의**: 도커 이미지를 공유하는 저장소
- **비유**: 앱스토어
- **주소**: https://hub.docker.com

### 4.2 이미지와 컨테이너 관계

```
📁 이미지 (ubuntu:22.04)
  │
  ├─ 📦 컨테이너1 (실행 중)
  ├─ 📦 컨테이너2 (중지됨)
  └─ 📦 컨테이너3 (실행 중)

# 하나의 이미지로 여러 컨테이너를 만들 수 있음!
```

---

## 5. 도커 기본 명령어

### 5.1 이미지 관련

#### 이미지 다운로드
```bash
# Docker Hub에서 이미지 가져오기
docker pull ubuntu:22.04
docker pull nginx:latest
```

#### 이미지 목록 보기
```bash
docker images

# 출력 예:
# REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
# ubuntu        22.04     27941809078c   2 weeks ago    77.8MB
# nginx         latest    605c77e624dd   3 weeks ago    141MB
```

#### 이미지 삭제
```bash
docker rmi ubuntu:22.04
# 또는 IMAGE ID로
docker rmi 27941809078c
```

### 5.2 컨테이너 관련

#### 컨테이너 실행
```bash
# 기본 실행
docker run ubuntu:22.04

# 백그라운드 실행 (-d: detached)
docker run -d nginx:latest

# 포트 연결 (-p: port)
docker run -d -p 8080:80 nginx:latest
# PC의 8080포트 → 컨테이너의 80포트

# 이름 지정 (--name)
docker run -d --name my-nginx nginx:latest

# 인터랙티브 모드 (-it: interactive + tty)
docker run -it ubuntu:22.04 /bin/bash
```

#### 컨테이너 목록 보기
```bash
# 실행 중인 컨테이너
docker ps

# 모든 컨테이너 (중지된 것 포함)
docker ps -a

# 출력 예:
# CONTAINER ID   IMAGE          STATUS         PORTS      NAMES
# a1b2c3d4e5f6   nginx:latest   Up 10 minutes  80/tcp     my-nginx
```

#### 컨테이너 시작/중지/재시작
```bash
# 중지
docker stop my-nginx
docker stop a1b2c3d4e5f6  # CONTAINER ID로도 가능

# 시작
docker start my-nginx

# 재시작
docker restart my-nginx
```

#### 컨테이너 삭제
```bash
# 단일 삭제
docker rm my-nginx

# 강제 삭제 (실행 중이어도)
docker rm -f my-nginx

# 모든 중지된 컨테이너 삭제
docker container prune
```

#### 컨테이너 접속
```bash
# 실행 중인 컨테이너에 접속
docker exec -it my-nginx /bin/bash

# 나오기
exit  # 또는 Ctrl+D
```

#### 컨테이너 로그 보기
```bash
# 로그 확인
docker logs my-nginx

# 실시간 로그 (-f: follow)
docker logs -f my-nginx

# 마지막 100줄
docker logs --tail 100 my-nginx
```

### 5.3 시스템 관리

#### 디스크 사용량 확인
```bash
docker system df

# 출력 예:
# TYPE            TOTAL     ACTIVE    SIZE
# Images          10        5         2.5GB
# Containers      15        3         1.2GB
```

#### 청소하기
```bash
# 사용하지 않는 이미지/컨테이너/네트워크 모두 삭제
docker system prune

# 더 강력한 청소 (볼륨 포함)
docker system prune -a --volumes
```

---

## 6. 두산 로봇에서 도커 사용하기

### 6.1 두산 에뮬레이터 설치

```bash
cd ~/ros2_ws/src/doosan-robot2
chmod +x ./install_emulator.sh
sudo ./install_emulator.sh
```

**이 스크립트가 하는 일:**
1. 도커 이미지 다운로드
2. 에뮬레이터 설정
3. 자동 실행 스크립트 설치

### 6.2 Virtual Mode 실행 과정

#### 방법 1: 자동 실행 (launch 파일이 알아서 처리)

```bash
# launch 파일 실행
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12345 \
    model:=m1013
```

**내부에서 일어나는 일:**
```bash
1. launch 파일이 도커 컨테이너 시작
   → docker run -d -p 12345:12345 doosan-emulator
2. 에뮬레이터가 127.0.0.1:12345 포트로 대기
3. ROS2 노드가 해당 포트로 연결
4. launch 종료 시 컨테이너 자동 종료
```

#### 방법 2: 수동 실행 (이해를 위해)

```bash
# 1. 에뮬레이터 컨테이너 실행
docker run -d \
    --name doosan-emulator \
    -p 12345:12345 \
    doosan-robotics/emulator:latest

# 2. 컨테이너가 실행 중인지 확인
docker ps | grep doosan-emulator

# 3. 로그 확인
docker logs -f doosan-emulator

# 4. ROS2 실행 (다른 터미널)
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual \
    host:=127.0.0.1 \
    port:=12345 \
    model:=m1013

# 5. 사용 후 정리
docker stop doosan-emulator
docker rm doosan-emulator
```

### 6.3 도커 상태 확인

#### 에뮬레이터가 실행 중인지 확인
```bash
# 실행 중인 컨테이너 찾기
docker ps | grep doosan

# 또는 모든 도커 컨테이너 보기
docker ps -a
```

#### 포트가 열려있는지 확인
```bash
# 12345 포트 확인
netstat -tuln | grep 12345

# 또는
ss -tuln | grep 12345

# 출력 예:
# tcp   LISTEN   0.0.0.0:12345
```

#### 에뮬레이터 로그 보기
```bash
# 실시간 로그
docker logs -f <container_id>

# 예시
docker logs -f a1b2c3d4e5f6
```

### 6.4 문제 해결

#### ❌ 문제: "Cannot connect to the Docker daemon"
```bash
# 원인: 도커 서비스가 실행 중이 아님
# 해결:
sudo systemctl start docker
sudo systemctl enable docker
```

#### ❌ 문제: "permission denied" 에러
```bash
# 원인: docker 그룹에 사용자가 없음
# 해결:
sudo usermod -aG docker $USER
# 로그아웃 후 다시 로그인
```

#### ❌ 문제: "port is already allocated"
```bash
# 원인: 12345 포트가 이미 사용 중
# 해결:

# 1. 해당 포트 사용 중인 컨테이너 찾기
docker ps | grep 12345

# 2. 컨테이너 중지
docker stop <container_id>

# 또는 포트 사용 중인 프로세스 종료
sudo lsof -i :12345
sudo kill -9 <PID>
```

#### ❌ 문제: "Connection refused to 127.0.0.1:12345"
```bash
# 확인 사항:
# 1. 컨테이너가 실행 중인가?
docker ps

# 2. 에뮬레이터 로그 확인
docker logs <container_id>

# 3. 컨테이너 재시작
docker restart <container_id>

# 4. 새로 시작
docker stop <container_id>
docker rm <container_id>
# launch 파일 다시 실행
```

---

## 7. 자주 묻는 질문

### Q1: 도커를 꼭 써야 하나요?
**A:** Virtual Mode를 사용하려면 필수입니다. Real Mode (실제 로봇)만 사용한다면 불필요합니다.

```
실제 로봇 있음 → Docker 불필요
실제 로봇 없음 → Docker 필요 (가상 로봇 사용)
```

### Q2: 도커가 무거운가요?
**A:** 가벼운 편입니다.

```
가상머신 (VMware, VirtualBox)
- OS 전체 실행: 2~4GB RAM
- 부팅 시간: 1~2분

도커
- 필요한 부분만 실행: 100~500MB
- 시작 시간: 1~5초
```

### Q3: 도커 컨테이너는 언제 종료되나요?
**A:** 두산 로봇 Virtual Mode의 경우, launch 파일 종료 시 자동으로 컨테이너도 종료됩니다.

```bash
# launch 파일 실행
ros2 launch ... mode:=virtual
# → 도커 컨테이너 자동 시작

# Ctrl+C로 launch 종료
# → 도커 컨테이너 자동 종료
```

### Q4: 여러 개의 가상 로봇을 동시에 실행할 수 있나요?
**A:** 가능합니다! 각각 다른 포트를 사용하면 됩니다.

```bash
# 로봇 1
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual port:=12345 name:=dsr01

# 로봇 2 (다른 터미널)
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py \
    mode:=virtual port:=12346 name:=dsr02
```

### Q5: 도커 이미지/컨테이너가 차지하는 공간이 너무 큽니다
**A:** 정기적으로 정리하세요.

```bash
# 사용하지 않는 것 모두 삭제
docker system prune -a

# 확인
docker system df
```

### Q6: 도커 없이 Virtual Mode를 사용할 수 없나요?
**A:** 이론적으로는 가능하지만, 매우 복잡합니다.

```
도커 없이:
1. 두산 에뮬레이터 소스코드 받기
2. 의존성 라이브러리 100개+ 수동 설치
3. 환경 변수 설정
4. 빌드
5. 실행

도커 사용:
./install_emulator.sh
끝!
```

### Q7: 실행 중인 컨테이너 안에서 명령어를 실행하려면?
**A:** `docker exec` 사용

```bash
# 컨테이너 내부 쉘 접속
docker exec -it <container_id> /bin/bash

# 한 줄 명령어 실행
docker exec <container_id> ls -la
docker exec <container_id> cat /etc/os-release
```

### Q8: 도커 컨테이너의 파일을 내 PC로 복사하려면?
**A:** `docker cp` 사용

```bash
# 컨테이너 → PC
docker cp <container_id>:/path/in/container /path/on/host

# PC → 컨테이너
docker cp /path/on/host <container_id>:/path/in/container

# 예시
docker cp my-container:/var/log/app.log ~/Desktop/
```

---

## 8. 도커 실전 팁

### 8.1 유용한 별칭 (Alias) 설정

`~/.bashrc`에 추가:

```bash
# 도커 단축 명령어
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dexec='docker exec -it'
alias dlogs='docker logs -f'
alias dclean='docker system prune -f'

# 적용
source ~/.bashrc
```

사용 예:
```bash
dps           # docker ps
dpsa          # docker ps -a
dexec my-container /bin/bash  # docker exec -it my-container /bin/bash
```

### 8.2 로그 파일 크기 제한

컨테이너 로그가 너무 커지는 것을 방지:

`/etc/docker/daemon.json` 생성/수정:
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

적용:
```bash
sudo systemctl restart docker
```

### 8.3 자주 사용하는 패턴

```bash
# 모든 컨테이너 중지
docker stop $(docker ps -q)

# 모든 컨테이너 삭제
docker rm $(docker ps -aq)

# 사용하지 않는 이미지 삭제
docker image prune -a

# 특정 이름의 컨테이너 찾아 중지
docker ps | grep "doosan" | awk '{print $1}' | xargs docker stop
```

---

## 9. 요약

### 도커를 한 문장으로

> **도커 = 가상의 독립적인 컴퓨터 환경을 만들어서 프로그램을 실행하는 도구**

### 두산 로봇에서 도커의 역할

```
실제 로봇 없이 → 도커로 가상 로봇 실행 → 내 코드 테스트
```

### 꼭 기억할 명령어

```bash
# 상태 확인
docker ps                  # 실행 중인 컨테이너
docker images              # 이미지 목록

# 컨테이너 관리
docker stop <id>           # 중지
docker start <id>          # 시작
docker rm <id>             # 삭제
docker logs -f <id>        # 로그 보기

# 청소
docker system prune        # 정리
```

### 문제 발생 시 체크리스트

- [ ] 도커 서비스가 실행 중인가? (`sudo systemctl status docker`)
- [ ] 사용자가 docker 그룹에 있는가? (`groups | grep docker`)
- [ ] 포트가 이미 사용 중인가? (`netstat -tuln | grep 12345`)
- [ ] 컨테이너가 실행 중인가? (`docker ps`)
- [ ] 에뮬레이터 로그에 에러가 있는가? (`docker logs <container_id>`)

---
 
**마지막 업데이트:** 2025년 12월 21일

**다음 단계:**
- [두산 로보틱스 ROS2 가이드](./DOOSAN_ROS2_GUIDE.md) 보기
- 실제로 Virtual Mode로 로봇 실행해보기
