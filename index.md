---
layout: page
title: Gosha Arinich
---

<p class="large">Hello! I'm a passionate and curious application developer.</p>

You can look at my code on [github](http://github.com/{{ site.author.github }}),
follow me on [twitter](http://twitter.com/{{ site.author.twitter }}), and
[email](mailto:{{ site.author.email }}) me.

{% if site.author.available_for_hire %}
[I am currently available for limited freelance work](mailto:{{ site.author.email }}).
{% endif %}

### Blog posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
