---
layout: post
title: Tags
permalink: /tags
---

{% for cat in site.categories %}
 <h2 id="{{ cat[0] }}">{{ cat[0] | capitalize }}</h2>
  
 {% for tag in cat.tags %}
  <h3 id="{{ tag[0] }}">{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
     {% if post.category contains cat[0] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
     {% endif %}
    {% endfor %}
  </ul>
 {% endfor %}
{% endfor %}
