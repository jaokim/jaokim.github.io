---
layout: post
title: Xena
permalink: /amiga/xena
---

Xena is the [XMOS XS-1](https://www.xmos.com/xs1) chip that is integrated on the AmigaOne [https://www.amigaos.net/hardware/35/amigaone-x1000](X1000) and [https://www.amigaos.net/hardware/133/amigaone-x5000](X5000) computers. In the posts below I share my efforts on doing something (useful?) with this chip.

{% for tag in site.tags %}
  <ul>
    {% if tag[0] contains "Xena" %}
     {% for post in tag[1] %}
       <li><a href="{{ post.url }}">{{ post.title }}</a></li>
       {% endfor %}
     {% endif %}
  </ul>
{% endfor %}
