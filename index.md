---
layout: default
---


 {% for post in site.posts %}
  {% if post.draft %}
    {% continue %}
  {% endif %}
  {% if post.categories contains "amiga" %}
    {% continue %}
  {% endif %}
  <article >
    <div class="index_article" onclick="location.href='{{ post.url }}'">
      <img class="index_img" src="{{post.image}}" width="35%">
      <div class="index_excerpt" width="65%">
      <h2>{{ post.title }}</h2>
      {{ post.excerpt }}
      <br/> <small>{% for pcat in post.categories %}cs {{pcat}} {% endfor %} </small>
      <br/> <small>{% for pcat in post.category %}ct {{pcat}} {% endfor %} </small>
      <br/><small><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></small>
      </div>
    </div>
    <hr>
  </article>
{% endfor %}
