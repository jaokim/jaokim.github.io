---
layout: post
title: Xena
permalink: /amiga/xena
---

 <h2>Xena</h2>

{% for tag in site.tags %}
  <ul>
     {% for post in tag[1] %}
       {% if post.tag contains "xena" %}
       <li><a href="{{ post.url }}">{{ post.title }}</a></li>
       {% endif %}
     {% endfor %}
  </ul>
{% endfor %}
