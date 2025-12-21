---
layout: archive
title: "Study Guides"
permalink: /study/
author_profile: true
---

<div class="page__hero">
  <div class="page__hero-inner">
    <p class="lead">ROS2 & Doosan Robot 학습 과정에서 정리한 실전 가이드. QoS 개념부터 Docker 환경 구축, 면접 대비까지.</p>
  </div>
</div>

<div class="collection-study">
  <div class="grid__wrapper">
  {% for post in site.study reversed %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
  </div>
</div>
