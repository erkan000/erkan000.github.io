---
title: Linux Notları
date: 12.01.2020
---

Linux notlarım


{% for bolum in site.linux %}

<li><a href="{{ bolum.url }}">{{ bolum.title }}</a></li>

{% endfor %}
