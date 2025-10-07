---
layout: post
title: Tags
permalink: /tags
---

{% for cat in site.categories %}
 <h2 id="{{ cat[0] }}">{{ cat[0] | capitalize }}</h2>
  
 {% for tag in site.tags %}
   {% assign current_tag = tag[0] %}
    {% for post in tag[1] %}
     {% if post.category contains cat[0] %}
      {% if current_tag != "" %}
  <h3 id="{{ tag[0] }}">{{ tag[0] }}</h3>
  <ul>
      {% endif %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% if current_tag != "" %}
  </ul>
      {% assign current_tag = "" %}
      {% endif %}
     {% endif %}
    {% endfor %}
 {% endfor %}
{% endfor %}
