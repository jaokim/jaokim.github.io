---

---


 {% for post in site.posts %}
  <article>
    <p>
        <a href="{{ post.url }}">
        <img src="{{post.splash-url}}" style="float:left; margin: 0px 10px 0px 0px; box-shadow: 0px 0px 10px 0px rgba(0,0,0,0.5);" width="35%"/>
        <h2>
            {{ post.title }}
        </h2>
        {{ post.excerpt }}
        </a>
        <br/><small><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></small>
    </p>
    <hr>
  </article>
{% endfor %}
