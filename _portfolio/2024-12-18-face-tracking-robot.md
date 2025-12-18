---
title: "Face Tracking Robot — Web Control System"
short_title: "Face-Tracking Project"
date: 2024-12-18
collection: portfolio
type: Project
permalink: /projects/face-tracking-robot/
teaser: homepage.png
excerpt: "ROS2 기반 얼굴 추적 로봇의 전체 아키텍처: ROS2 노드, Web UI, IPC, 비전·음성 파이프라인을 통합한 실전형 설계 문서."
read_time: true
share: true
categories: [Project, Robotics]
tags: [ROS2, Face Tracking, Web, Doosan, RealSense]
---

# Face Tracking Robot — Web Control System (v2.0)

System: ROS2 Humble · Doosan M0609 · OnRobot RG2 · RealSense D435i

Date: 2024-12-18

이 페이지는 얼굴 추적 기반 로봇 웹 제어 시스템의 설계 문서이자 포트폴리오 항목입니다. ROS2 기반 로봇 제어, 웹 UI, 파일 기반 IPC, 비전·음성·시나리오 파이프라인까지 실제 시연 가능한 수준의 아키텍처를 정리합니다.

---

## 핵심 기능

- 통합 URDF 및 `ros2_control` 기반 로봇 제어
- RG2 그리퍼 Modbus 제어 + RViz 동기화
- RealSense → MediaPipe → Face Tracking → Joint Tracking 파이프라인
- ROS2 ↔ Web 브리지(상태 JSON, 명령 파일 기반 IPC, WebSocket 비디오)
- FastAPI 백엔드 + React 프런트엔드의 실시간 제어 인터페이스

---

## 아키텍처 요약

프로젝트 구조 및 주요 노드, 네트워크 맵, IPC 및 시나리오 상태 머신을 포함하여 실전 운영을 목표로 설계했습니다. 하드웨어/소프트웨어 패키지 목록, 네트워크 구성, 로그/테스트 명령 등도 문서화되어 있어 배포·시연에 바로 활용 가능합니다.

더 자세한 문서는 저장소 내 README 및 개별 패키지 문서를 참고하세요.
