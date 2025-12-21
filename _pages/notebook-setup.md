---
layout: archive
title: "노트북 설정"
permalink: /notebook-setup/
author_profile: true
---

<div class="page__hero">
  <div class="page__hero-inner">
    <p class="lead">개발 노트북 환경 구축 및 최적화 가이드. GPU 설정부터 시스템 튜닝까지.</p>
  </div>
</div>

<div class="collection-notebook-setup">
  <div class="grid__wrapper">
  {% for post in site.notebook_setup reversed %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
  </div>
</div>
