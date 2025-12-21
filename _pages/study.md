---
layout: archive
title: "Study Guides"
permalink: /study/
author_profile: true
---

## ROS2 & 로봇 공학 학습 가이드

두산 로봇과 ROS2를 공부하면서 정리한 가이드 모음입니다.

### 🤖 ROS2 핵심 개념

{% for post in site.study %}
  {% if post.title contains "ROS2" %}
  {% include archive-single.html %}
  {% endif %}
{% endfor %}

### 🐳 Docker & 환경 설정

{% for post in site.study %}
  {% if post.title contains "Docker" or post.title contains "GPU" %}
  {% include archive-single.html %}
  {% endif %}
{% endfor %}

### 🦾 두산 로봇

{% for post in site.study %}
  {% if post.title contains "두산" %}
  {% include archive-single.html %}
  {% endif %}
{% endfor %}

---

*학습하면서 지속적으로 업데이트 중입니다.*
