---
title: Hello World
date: 24.03.2019
---

Hoşgeldin, burada hızlı notlarımı bulabilirsin.

<div>
{% for bolum in site.collections %}
	{% if bolum.label != 'posts' %}
		<h2><b>{{ bolum.label }}</b> klasörü</h2>
	  	<ul>
	    	{% for item in site[bolum.label] %}
	    		<li>  <a id="writelinklist" href="{{ item.url }}">{{ item.title }}</a></li>
	    	{% endfor %}
	  	</ul>
	  	<br>
	{% endif %}
{% endfor %}
</div>

<div>
	<a href="http://busrauzun.com/clean-code-kitabindan-notlar.html">Robert C. Martin, Clean Code kitabı notları!</a>
</div>
