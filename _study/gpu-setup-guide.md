---
title: "Intel Arc GPU 설정 가이드"
collection: study
type: "Guide"
permalink: /study/gpu-setup-guide
date: 2025-12-21
excerpt: "Ubuntu에서 Intel Arc Graphics 설정 및 최적화. Mesa 드라이버 설치, OpenGL 설정, 성능 모니터링."
---

# GPU 설정 및 관리 가이드

> Ubuntu 22.04 + Intel Arc Graphics 기준  
> 작성일: 2025년 12월 21일

---

## 📋 목차

1. [GPU 확인하기](#1-gpu-확인하기)
2. [GPU 드라이버 설치](#2-gpu-드라이버-설치)
3. [GPU 활성화/비활성화](#3-gpu-활성화비활성화)
4. [성능 모니터링](#4-성능-모니터링)
5. [문제 해결](#5-문제-해결)

---

## 1. GPU 확인하기

### 1.1 하드웨어 GPU 목록 보기

```bash
# PCI 장치에서 VGA 컨트롤러 찾기
lspci | grep -i vga

# 출력 예시:
# 00:02.0 VGA compatible controller: Intel Corporation Meteor Lake-P [Intel Arc Graphics] (rev 08)
```

### 1.2 현재 사용 중인 GPU 확인

```bash
# OpenGL 정보 확인 (mesa-utils 필요)
glxinfo | grep "OpenGL renderer"

# 출력 예시:
# OpenGL renderer string: Mesa Intel(R) Arc(tm) Graphics (MTL)
```

### 1.3 상세 GPU 정보

```bash
# OpenGL 버전 및 렌더러
glxinfo | grep -E "OpenGL version|OpenGL renderer|Direct rendering"

# 출력 예시:
# Direct rendering: Yes
# OpenGL renderer string: Mesa Intel(R) Arc(tm) Graphics (MTL)
# OpenGL version string: 4.6 (Compatibility Profile) Mesa 23.2.1
```

### 1.4 GPU 드라이버 정보

```bash
# Intel GPU 정보 상세
sudo lshw -C display

# 또는
lspci -v | grep -A 20 VGA
```

---

## 2. GPU 드라이버 설치

### 2.1 필수 도구 설치

```bash
# mesa-utils 설치 (glxinfo 등)
sudo apt update
sudo apt install -y mesa-utils

# 추가 GPU 모니터링 도구
sudo apt install -y intel-gpu-tools
```

### 2.2 Intel Arc Graphics 드라이버

#### 자동 설치 (Ubuntu 22.04)

```bash
# 기본 Mesa 드라이버 (이미 설치되어 있음)
sudo apt install -y mesa-va-drivers mesa-vulkan-drivers
```

#### 최신 드라이버 설치 (선택)

```bash
# Intel 공식 저장소 추가
wget -qO - https://repositories.intel.com/gpu/intel-graphics.key | \
  sudo gpg --dearmor --output /usr/share/keyrings/intel-graphics.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu jammy client" | \
  sudo tee /etc/apt/sources.list.d/intel-gpu-jammy.list

# 업데이트 및 설치
sudo apt update
sudo apt install -y intel-opencl-icd intel-level-zero-gpu level-zero intel-media-va-driver-non-free
```

### 2.3 설치 확인

```bash
# OpenGL 테스트
glxinfo | head -20

# Vulkan 지원 확인
vulkaninfo | grep "deviceName"

# Intel GPU 도구 확인
intel_gpu_top --help
```

---

## 3. GPU 활성화/비활성화

### 3.1 GPU 상태 확인

```bash
# 현재 GPU 전원 상태
cat /sys/class/drm/card0/device/power/runtime_status

# 출력:
# active   (활성)
# suspended (절전 모드)
```

### 3.2 GPU 절전 모드 설정

#### 자동 절전 모드 설정

```bash
# 절전 모드 타임아웃 설정 (밀리초, -1은 비활성화)
sudo sh -c 'echo "auto" > /sys/class/drm/card0/device/power/control'

# 타임아웃 시간 설정 (예: 5초)
sudo sh -c 'echo 5000 > /sys/class/drm/card0/device/power/autosuspend_delay_ms'
```

#### 절전 모드 비활성화 (항상 활성)

```bash
# GPU 항상 켜진 상태 유지
sudo sh -c 'echo "on" > /sys/class/drm/card0/device/power/control'
```

### 3.3 GPU 완전 비활성화 (사용 안 함)

⚠️ **주의**: 일반적으로 권장하지 않습니다!

```bash
# 방법 1: 커널 파라미터로 비활성화
sudo nano /etc/default/grub

# 다음 줄 수정:
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
# 를
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i915.modeset=0"

# GRUB 업데이트
sudo update-grub

# 재부팅
sudo reboot
```

### 3.4 GPU 다시 활성화

```bash
# 방법 1: 커널 파라미터 제거
sudo nano /etc/default/grub
# i915.modeset=0 제거

sudo update-grub
sudo reboot

# 방법 2: 드라이버 재로드
sudo modprobe -r i915
sudo modprobe i915
```

---

## 4. 성능 모니터링

### 4.1 실시간 GPU 사용량 확인

```bash
# Intel GPU Top (실시간 모니터링)
sudo intel_gpu_top

# 출력 예시:
# intel-gpu-top - 1234/ 1234 MHz;  45% RC6;  2.50 Watts;     1234 irqs/s
# 
#       ENGINES     BUSY
#        Render:    85.0%
#         Video:     0.0%
#         Blitter:   0.0%
```

### 4.2 GPU 온도 확인

```bash
# sensors 도구 설치
sudo apt install -y lm-sensors

# 센서 감지
sudo sensors-detect

# 온도 확인
sensors | grep -A 5 "Package"
```

### 4.3 GPU 메모리 사용량

```bash
# Intel GPU 메모리 정보
cat /sys/kernel/debug/dri/0/i915_gem_objects

# 또는
intel_gpu_top -l
```

### 4.4 GPU 주파수 확인

```bash
# 현재 GPU 주파수
cat /sys/class/drm/card0/gt_cur_freq_mhz

# 최소 주파수
cat /sys/class/drm/card0/gt_min_freq_mhz

# 최대 주파수
cat /sys/class/drm/card0/gt_max_freq_mhz
```

---

## 5. 문제 해결

### 5.1 GPU가 인식되지 않을 때

#### 문제 확인
```bash
# GPU 장치 확인
ls /dev/dri/

# 출력에 card0, renderD128 등이 보여야 함
# card0       - GPU 디스플레이
# renderD128  - GPU 렌더링
```

#### 해결 방법

**1. 드라이버 재설치**
```bash
sudo apt install --reinstall xserver-xorg-video-intel mesa-utils
sudo reboot
```

**2. 커널 업데이트**
```bash
# 최신 커널 설치
sudo apt update
sudo apt install linux-generic-hwe-22.04
sudo reboot
```

**3. 펌웨어 업데이트**
```bash
sudo apt install intel-microcode
sudo update-initramfs -u
sudo reboot
```

### 5.2 GPU 성능이 낮을 때

#### 전원 관리 설정 확인
```bash
# 성능 모드로 변경
sudo sh -c 'echo "performance" > /sys/class/drm/card0/device/power_dpm_force_performance_level'

# 또는 자동
sudo sh -c 'echo "auto" > /sys/class/drm/card0/device/power_dpm_force_performance_level'
```

#### GPU 주파수 제한 해제
```bash
# 최대 주파수 확인
cat /sys/class/drm/card0/gt_max_freq_mhz

# 부스트 주파수 설정 (예: 2000MHz)
sudo sh -c 'echo 2000 > /sys/class/drm/card0/gt_boost_freq_mhz'
```

### 5.3 RViz2에서 GPU 사용 안 될 때

#### 환경 변수 설정
```bash
# ~/.bashrc에 추가
export LIBGL_ALWAYS_SOFTWARE=0
export MESA_LOADER_DRIVER_OVERRIDE=iris

# 적용
source ~/.bashrc
```

#### 직접 렌더링 확인
```bash
glxinfo | grep "direct rendering"

# Yes가 나와야 GPU 사용 중
# Direct rendering: Yes
```

### 5.4 "Could not initialize GLX" 에러

```bash
# OpenGL 라이브러리 재설치
sudo apt install --reinstall libgl1-mesa-glx libgl1-mesa-dri

# 권한 문제 해결
sudo usermod -aG video $USER
sudo usermod -aG render $USER

# 로그아웃 후 다시 로그인
```

### 5.5 화면 깜빡임/티어링 문제

#### Intel TearFree 옵션 활성화

```bash
# X11 설정 파일 생성
sudo nano /etc/X11/xorg.conf.d/20-intel.conf

# 다음 내용 추가:
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option      "TearFree" "true"
EndSection

# 재시작
sudo reboot
```

---

## 6. 유용한 스크립트

### 6.1 GPU 상태 확인 스크립트

`~/gpu_status.sh` 생성:

```bash
#!/bin/bash

echo "=== GPU 정보 ==="
lspci | grep -i vga

echo -e "\n=== 드라이버 ==="
glxinfo | grep "OpenGL renderer"
glxinfo | grep "OpenGL version"

echo -e "\n=== 전원 상태 ==="
cat /sys/class/drm/card0/device/power/runtime_status

echo -e "\n=== GPU 주파수 ==="
echo "현재: $(cat /sys/class/drm/card0/gt_cur_freq_mhz) MHz"
echo "최소: $(cat /sys/class/drm/card0/gt_min_freq_mhz) MHz"
echo "최대: $(cat /sys/class/drm/card0/gt_max_freq_mhz) MHz"
```

실행 권한 부여 및 실행:
```bash
chmod +x ~/gpu_status.sh
~/gpu_status.sh
```

### 6.2 GPU 성능 모드 전환 스크립트

`~/gpu_performance.sh` 생성:

```bash
#!/bin/bash

if [ "$1" == "high" ]; then
    echo "고성능 모드 활성화..."
    sudo sh -c 'echo "on" > /sys/class/drm/card0/device/power/control'
    echo "완료!"
elif [ "$1" == "auto" ]; then
    echo "자동 절전 모드 활성화..."
    sudo sh -c 'echo "auto" > /sys/class/drm/card0/device/power/control'
    echo "완료!"
else
    echo "사용법: $0 [high|auto]"
    echo "  high - 고성능 모드 (항상 켜짐)"
    echo "  auto - 자동 절전 모드"
fi
```

사용:
```bash
chmod +x ~/gpu_performance.sh

# 고성능 모드
~/gpu_performance.sh high

# 자동 모드
~/gpu_performance.sh auto
```

---

## 7. 벤치마크 및 테스트

### 7.1 OpenGL 벤치마크

```bash
# glmark2 설치
sudo apt install -y glmark2

# 벤치마크 실행
glmark2

# 풀스크린 벤치마크
glmark2 --fullscreen
```

### 7.2 간단한 GPU 테스트

```bash
# glxgears (간단한 FPS 테스트)
glxgears

# VSync 비활성화로 최대 성능 테스트
vblank_mode=0 glxgears
```

### 7.3 Vulkan 테스트

```bash
# vulkan-tools 설치
sudo apt install -y vulkan-tools

# Vulkan 정보
vulkaninfo

# Vulkan 큐브 데모
vkcube
```

---

## 8. 권장 설정 (ROS2 + RViz2 사용 시)

### 8.1 최적 설정

`~/.bashrc`에 추가:

```bash
# GPU 가속 활성화
export LIBGL_ALWAYS_SOFTWARE=0

# Intel Arc 드라이버 사용
export MESA_LOADER_DRIVER_OVERRIDE=iris

# VSync 설정 (티어링 방지)
export vblank_mode=1

# 적용
source ~/.bashrc
```

### 8.2 RViz2 실행 전 체크리스트

- [ ] GPU가 인식되는가? (`lspci | grep -i vga`)
- [ ] 직접 렌더링이 활성화되었는가? (`glxinfo | grep "direct rendering"`)
- [ ] OpenGL 버전이 4.x 이상인가? (`glxinfo | grep "OpenGL version"`)
- [ ] GPU 전원이 활성 상태인가? (`cat /sys/class/drm/card0/device/power/runtime_status`)

### 8.3 성능 향상 팁

```bash
# 1. 고성능 모드 유지
sudo sh -c 'echo "on" > /sys/class/drm/card0/device/power/control'

# 2. 최대 주파수로 설정
MAX_FREQ=$(cat /sys/class/drm/card0/gt_max_freq_mhz)
sudo sh -c "echo $MAX_FREQ > /sys/class/drm/card0/gt_boost_freq_mhz"

# 3. RViz2 실행 시 우선순위 높이기
nice -n -10 ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=virtual model:=m1013
```

---

## 9. 요약

### GPU 켜기 (활성화)

```bash
# 방법 1: 자동 관리
sudo sh -c 'echo "auto" > /sys/class/drm/card0/device/power/control'

# 방법 2: 항상 켜짐 (고성능)
sudo sh -c 'echo "on" > /sys/class/drm/card0/device/power/control'
```

### GPU 끄기 (절전 모드)

```bash
# 방법 1: 자동 절전
sudo sh -c 'echo "auto" > /sys/class/drm/card0/device/power/control'
sudo sh -c 'echo 100 > /sys/class/drm/card0/device/power/autosuspend_delay_ms'

# 방법 2: 강제 절전 (비권장)
sudo sh -c 'echo "suspend" > /sys/class/drm/card0/device/power/control'
```

### 상태 확인

```bash
# 한 줄로 모든 정보 확인
echo "GPU: $(lspci | grep -i vga | cut -d: -f3)" && \
echo "드라이버: $(glxinfo | grep "OpenGL renderer" | cut -d: -f2)" && \
echo "전원: $(cat /sys/class/drm/card0/device/power/runtime_status)" && \
echo "주파수: $(cat /sys/class/drm/card0/gt_cur_freq_mhz) MHz"
```

---

**작성자:** GitHub Copilot  
**마지막 업데이트:** 2025년 12월 21일  
**GPU:** Intel Arc Graphics (Meteor Lake)

**관련 가이드:**
- [두산 로보틱스 ROS2 가이드](./DOOSAN_ROS2_GUIDE.md)
- [도커 완전 초보 가이드](./DOCKER_GUIDE.md)
