---
title: Sistem Notları
date: 24.03.2019
---

Sistem notları


{% for bolum in site.sistem %}

<li><a href="{{ bolum.url }}">{{ bolum.title }}</a></li>

{% endfor %}

