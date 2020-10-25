---
title: Sosyal
date: 09.02.2020
---

Edebiyat, tarih ve çeşitli yazıları bulunmaktadır.


{% for bolum in site.sosyal %}

<li><a href="{{ bolum.url }}">{{ bolum.title }}</a></li>

{% endfor %}

