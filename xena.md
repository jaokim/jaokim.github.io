---
layout: post
title: Tags
permalink: /tags
---

 <h2>Xena</h2>

{% for tag in site.tags %}
  <ul>
     {% for post in tag[1] %}
       {% if post.tags contains "xena" %}
       <li><a href="{{ post.url }}">{{ post.title }}</a></li>
       {% endif %}
     {% endfor %}
  </ul>
{% endfor %}
