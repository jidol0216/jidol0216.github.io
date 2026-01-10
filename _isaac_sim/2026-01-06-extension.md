---
title: "Isaac Sim Extension 활용"
collection: isaac_sim
date: 2026-01-06
permalink: /isaac-sim/extension/
excerpt: "Isaac Sim에서 Extension을 활용해 GUI 통합, 반복 작업 자동화, 배포를 쉽게 하는 방법과 예시들."
---

## Isaac Sim Extension 활용

### Extension을 쓰는 이유

1. **GUI 통합** - Isaac Sim 안에서 버튼 클릭으로 작업 가능
2. **반복 작업 자동화** - 버튼 한 번으로 복잡한 씬 구성
3. **배포 용이** - Extension 폴더만 주면 같은 환경 재현 가능

### Extension 활용 예시

| 활용 | 설명 |
|------|------|
| 실시간 제어 UI | 슬라이더로 로봇 속도 조절 |
| 로봇 스폰 도구 | 드롭다운에서 로봇 선택 후 스폰 |
| 디버그 정보 표시 | 실시간 센서 데이터, 속도 그래프 |
| Waypoint 에디터 | 클릭으로 경로점 추가 |
| 환경 생성 도구 | 장애물 랜덤 배치 |
| 데이터 수집 | 학습용 데이터셋 자동 저장 |
| 파라미터 튜닝 | Physics 파라미터 실시간 조정 |

### Extension 없이도 되는 것들

- 씬 빌드/저장 → Python 스크립트로 가능
- ROS2 연동 → 기본 제공
- 시뮬레이션 실행 → standalone script로 가능

**Extension이 필요한 경우**: 커스텀 UI, 실시간 조작 패널, 메뉴/버튼 추가
