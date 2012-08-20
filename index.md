---
layout: page
title: Gosha Arinich
---

Hi, I'm Gosha Arinich. I'm an young geek and passionate application
developer. I like to [code](http://github.com/{{ site.author.github }})
and receive [emails](mailto:{{ site.author.email }}) from awesome people.

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
