---
title: "리눅스 필수 명령어 모음"
collection: linux
type: "Reference"
permalink: /linux/linux-commands
date: 2025-12-21
excerpt: "로봇 개발과 ROS2 환경 구축에 필요한 리눅스 명령어 완벽 정리. 네트워크, 프로세스, 파일 관리 등 실전 예제 포함."
---

# 리눅스 필수 명령어 모음

로봇 개발과 ROS2 환경 구축에서 자주 사용하는 리눅스 명령어들을 정리했습니다.

## 📡 네트워크 명령어

### IP 주소 관리
```bash
# IP 주소 확인
ip addr show
ip a

# 특정 인터페이스에 IP 추가
sudo ip addr add 192.168.1.50/24 dev enxc84d442343d5

# IP 주소 삭제
sudo ip addr del 192.168.1.50/24 dev enxc84d442343d5

# 네트워크 인터페이스 활성화/비활성화
sudo ip link set enxc84d442343d5 up
sudo ip link set enxc84d442343d5 down
```

### 네트워크 연결 테스트
```bash
# 핑 테스트
ping 192.168.1.100
ping -c 4 192.168.1.100  # 4번만 전송

# 포트 확인
netstat -tuln | grep 8080
ss -tuln | grep 8080

# DNS 조회
nslookup google.com
dig google.com
```

### ifconfig (구형 명령어)
```bash
# 네트워크 인터페이스 확인
ifconfig
ifconfig enxc84d442343d5

# IP 주소 설정 (권장하지 않음, ip 명령어 사용 권장)
sudo ifconfig enxc84d442343d5 192.168.1.50 netmask 255.255.255.0
```

---

## 🤖 ROS2 명령어

### ROS2 기본
```bash
# ROS2 도메인 설정
export ROS_DOMAIN_ID=64
echo $ROS_DOMAIN_ID

# ROS2 패키지 실행
ros2 launch dsr_launcher2 single_robot_rviz.launch.py

# 백그라운드 실행
ros2 launch dsr_launcher2 single_robot_rviz.launch.py &
```

### Topic 관련
```bash
# 토픽 목록 확인
ros2 topic list

# 토픽 상세 정보
ros2 topic info /dsr01/joint_states

# 토픽 타입 확인
ros2 topic type /dsr01/joint_states

# 토픽 메시지 모니터링
ros2 topic echo /dsr01/joint_states

# 토픽으로 메시지 발행
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 1.0}}"
```

### Service 관련
```bash
# 서비스 목록
ros2 service list

# 서비스 타입 확인
ros2 service type /dsr01/motion/move_joint

# 서비스 호출
ros2 service call /dsr01/motion/move_home std_srvs/srv/Trigger

# 서비스 인터페이스 확인
ros2 interface show dsr_msgs2/srv/MoveJoint
```

### Node 관련
```bash
# 실행 중인 노드 확인
ros2 node list

# 노드 정보 확인
ros2 node info /dsr01_controller

# 노드 간 연결 관계
ros2 node info /dsr01_controller --verbose
```

### 패키지 관리
```bash
# 패키지 찾기
ros2 pkg list
ros2 pkg list | grep dsr

# 패키지 경로 확인
ros2 pkg prefix dsr_launcher2

# 실행 파일 찾기
ros2 pkg executables dsr_launcher2
```

---

## 📁 파일 및 디렉토리 관리

### 기본 파일 조작
```bash
# 파일 복사
cp file.txt backup.txt
cp -r dir1/ dir2/  # 디렉토리 복사

# 파일 이동/이름 변경
mv file.txt new_name.txt
mv file.txt /path/to/destination/

# 파일 삭제
rm file.txt
rm -rf directory/  # 디렉토리 강제 삭제 (주의!)

# 디렉토리 생성
mkdir new_folder
mkdir -p parent/child/grandchild  # 중간 경로 자동 생성
```

### 파일 내용 확인
```bash
# 파일 전체 내용
cat file.txt

# 페이지 단위로 보기
less file.txt
more file.txt

# 파일 시작 부분
head file.txt
head -n 20 file.txt  # 처음 20줄

# 파일 끝 부분
tail file.txt
tail -f log.txt  # 실시간 모니터링

# 파일 내용 검색
grep "keyword" file.txt
grep -r "keyword" ./  # 디렉토리 재귀 검색
grep -i "keyword" file.txt  # 대소문자 무시
```

### 디렉토리 탐색
```bash
# 현재 위치
pwd

# 디렉토리 이동
cd /path/to/directory
cd ~  # 홈 디렉토리
cd -  # 이전 디렉토리

# 디렉토리 내용 보기
ls
ls -la  # 숨김 파일 포함, 상세 정보
ls -lh  # 파일 크기 읽기 쉽게
tree  # 트리 구조로 보기
tree -L 2  # 깊이 2단계까지
```

### 파일 찾기
```bash
# 파일명으로 찾기
find . -name "*.py"
find /path -name "robot_*"

# 타입으로 찾기
find . -type f  # 파일만
find . -type d  # 디렉토리만

# 시간으로 찾기
find . -mtime -7  # 최근 7일 이내 수정된 파일

# 찾은 파일에 명령 실행
find . -name "*.pyc" -delete
```

---

## ⚙️ 환경 변수 및 설정

### 환경 변수 관리
```bash
# 환경 변수 확인
echo $PATH
echo $ROS_DOMAIN_ID
env  # 모든 환경 변수
printenv

# 환경 변수 설정 (현재 세션만)
export ROS_DOMAIN_ID=64
export MY_VAR="value"

# 환경 변수 해제
unset MY_VAR

# .bashrc에 영구 등록
echo 'export ROS_DOMAIN_ID=64' >> ~/.bashrc
source ~/.bashrc
```

### 소스 파일 로드
```bash
# ROS2 환경 설정
source /opt/ros/humble/setup.bash

# 워크스페이스 빌드 후
source ~/ros2_ws/install/setup.bash

# .bashrc 다시 로드
source ~/.bashrc
```

### Alias 설정
```bash
# 임시 alias 설정
alias ll='ls -la'
alias ros2_env='source /opt/ros/humble/setup.bash'

# .bashrc에 영구 등록
echo "alias robot_start='ros2 launch dsr_launcher2 single_robot_rviz.launch.py'" >> ~/.bashrc
```

---

## 🔐 권한 및 프로세스 관리

### 파일 권한
```bash
# 권한 확인
ls -l file.txt

# 실행 권한 추가
chmod +x script.sh

# 권한 상세 설정
chmod 755 script.sh  # rwxr-xr-x
chmod 644 file.txt   # rw-r--r--

# 소유자 변경
sudo chown user:group file.txt
sudo chown -R user:group directory/
```

### 프로세스 관리
```bash
# 프로세스 목록
ps aux
ps aux | grep python

# 프로세스 트리
pstree

# 실시간 프로세스 모니터
top
htop  # 더 보기 좋음

# 프로세스 종료
kill PID
kill -9 PID  # 강제 종료
killall python3  # 이름으로 종료
pkill -f "ros2 launch"  # 패턴으로 종료

# 백그라운드 실행
command &
nohup command &  # 터미널 종료 후에도 계속 실행

# 백그라운드 작업 관리
jobs
fg %1  # 작업 1을 포그라운드로
bg %1  # 작업 1을 백그라운드로
```

---

## 📦 패키지 관리

### APT (Ubuntu/Debian)
```bash
# 패키지 업데이트
sudo apt update
sudo apt upgrade

# 패키지 설치
sudo apt install tree
sudo apt install -y net-tools  # -y는 자동 yes

# 패키지 제거
sudo apt remove package-name
sudo apt autoremove  # 불필요한 의존성 제거

# 패키지 검색
apt search keyword
apt show package-name
```

### Python 패키지
```bash
# pip 설치
pip install package-name
pip install -r requirements.txt

# pip 업그레이드
pip install --upgrade package-name

# 설치된 패키지 목록
pip list
pip freeze > requirements.txt
```

---

## 💾 디스크 및 시스템 정보

### 디스크 사용량
```bash
# 전체 디스크 사용량
df -h

# 디렉토리별 사용량
du -sh *
du -h --max-depth=1

# 특정 디렉토리 크기
du -sh /path/to/directory
```

### 시스템 정보
```bash
# 시스템 정보
uname -a
lsb_release -a

# CPU 정보
lscpu
cat /proc/cpuinfo

# 메모리 정보
free -h
cat /proc/meminfo

# GPU 정보
lspci | grep -i vga
nvidia-smi  # NVIDIA GPU
```

---

## 🔍 텍스트 처리

### grep (검색)
```bash
# 기본 검색
grep "pattern" file.txt

# 여러 옵션
grep -i "pattern" file.txt  # 대소문자 무시
grep -v "pattern" file.txt  # 패턴 제외
grep -n "pattern" file.txt  # 줄 번호 표시
grep -r "pattern" ./        # 재귀 검색
grep -c "pattern" file.txt  # 매칭 라인 수
```

### sed (편집)
```bash
# 문자열 치환
sed 's/old/new/' file.txt
sed 's/old/new/g' file.txt  # 모든 매칭 치환

# 특정 줄 삭제
sed '5d' file.txt
sed '/pattern/d' file.txt

# 파일 직접 수정
sed -i 's/old/new/g' file.txt
```

### awk (텍스트 처리)
```bash
# 특정 열 출력
awk '{print $1}' file.txt
ps aux | awk '{print $2, $11}'  # PID와 명령어

# 조건부 출력
awk '$3 > 50' file.txt  # 3번째 열이 50 이상

# 합계 계산
awk '{sum += $1} END {print sum}' file.txt
```

---

## 🔄 압축 및 아카이브

### tar
```bash
# 압축
tar -czf archive.tar.gz directory/
tar -czvf archive.tar.gz file1 file2

# 압축 해제
tar -xzf archive.tar.gz
tar -xzf archive.tar.gz -C /destination/

# 내용 확인
tar -tzf archive.tar.gz
```

### zip/unzip
```bash
# 압축
zip archive.zip file1 file2
zip -r archive.zip directory/

# 압축 해제
unzip archive.zip
unzip archive.zip -d /destination/
```

---

## 🌐 원격 접속

### SSH
```bash
# 원격 접속
ssh user@hostname
ssh user@192.168.1.100

# 포트 지정
ssh -p 2222 user@hostname

# SSH 키 생성
ssh-keygen -t rsa -b 4096

# 공개키 복사
ssh-copy-id user@hostname
```

### SCP (파일 전송)
```bash
# 로컬 → 원격
scp file.txt user@hostname:/path/

# 원격 → 로컬
scp user@hostname:/path/file.txt ./

# 디렉토리 전송
scp -r directory/ user@hostname:/path/
```

---

## 🛠️ 유용한 팁

### 명령어 히스토리
```bash
# 히스토리 확인
history

# 최근 명령어 다시 실행
!!

# 특정 명령어 검색 (Ctrl+R)
# 터미널에서 Ctrl+R 누르고 검색어 입력

# 특정 번호 명령어 실행
!123
```

### 파이프와 리다이렉션
```bash
# 파이프로 연결
ps aux | grep python | awk '{print $2}'

# 출력 리다이렉션
command > output.txt   # 덮어쓰기
command >> output.txt  # 추가

# 에러 리다이렉션
command 2> error.txt
command 2>&1 > all.txt  # 표준출력과 에러 모두
```

### 명령어 조합
```bash
# 순차 실행
command1 ; command2

# AND 조건 (첫 번째 성공시만 두 번째 실행)
command1 && command2

# OR 조건 (첫 번째 실패시만 두 번째 실행)
command1 || command2
```

---

## 📚 자주 사용하는 명령어 조합

### ROS2 로봇 시작 절차
```bash
# 1. 환경 변수 설정
export ROS_DOMAIN_ID=64

# 2. ROS2 환경 로드
source /opt/ros/humble/setup.bash

# 3. 로봇 런치
ros2 launch dsr_launcher2 single_robot_rviz.launch.py

# 4. 별도 터미널에서 제어 스크립트 실행
python3 robot_move_service.py
```

### 네트워크 문제 해결
```bash
# 1. 인터페이스 확인
ip addr show

# 2. IP 설정
sudo ip addr add 192.168.1.50/24 dev enxc84d442343d5

# 3. 연결 테스트
ping 192.168.1.100

# 4. ROS2 토픽 확인
ros2 topic list
```

### 로그 분석
```bash
# 실시간 로그 모니터링
tail -f ~/.ros/log/latest/ros.log

# 에러 메시지만 추출
grep -i error log.txt

# 타임스탬프 기준 최근 100줄
tail -n 100 log.txt | grep "2025-12-21"
```

---

## 🎯 키보드 단축키

```
Ctrl + C  : 현재 실행 중인 프로세스 종료
Ctrl + Z  : 현재 프로세스 일시정지 (bg로 재개 가능)
Ctrl + D  : EOF 전송 (터미널 종료)
Ctrl + L  : 화면 지우기 (clear와 동일)
Ctrl + R  : 명령어 히스토리 검색
Ctrl + A  : 줄 맨 앞으로
Ctrl + E  : 줄 맨 뒤로
Ctrl + U  : 커서 앞 내용 삭제
Ctrl + K  : 커서 뒤 내용 삭제
Tab       : 자동완성
```

---
