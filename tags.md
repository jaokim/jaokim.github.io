---
layout: post
title: Tags
permalink: /tags
---

{% for cat in site.categories %}
 <h2 id="{{ cat[0] }}">{{ cat[0] | capitalize }}</h2>
 {% for tag in site.tags %}
   {% assign tag_open = true %}
   {% assign tag_close = false %}
   {% assign current_tag = tag[0] %}
     {% for post in tag[1] %}
       {% if post.category contains cat[0] %}
         {% if tag_open %}
           {% assign tag_open = false %}
           {% assign tag_close = true %}
   
  <h3 id="{{ current_tag }}">{{ current_tag }}</h3>
  <ul>
         {% endif %}
       <li><a href="{{ post.url }}">{{ post.title }}</a></li>
       {% endif %}
     {% endfor %}
     {% if tag_close "" %}
  </ul>
       {% assign tag_close = false %}
     {% endif %}
 {% endfor %}
{% endfor %}
