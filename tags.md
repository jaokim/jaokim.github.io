---
layout: post
title: Tags
permalink: /tags
---

{% for cat in site.categories %}
 <h2 id="{{ cat }}">{{ cat }}</h3>
  
 {% for tag in site.tags %}
  <h3 id="{{ tag[0] }}">{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
     {% if post.category contains cat %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
     {% endif %}
    {% endfor %}
  </ul>
 {% endfor %}
{% endfor %}
