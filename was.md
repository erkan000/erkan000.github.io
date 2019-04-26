---
title: WAS Notları
date: 24.03.2019
---

was ile ilgili sayfa dır


{% for bolum in site.was %}

<li><a href="{{ bolum.url }}">{{ bolum.title }}</a></li>

{% endfor %}
