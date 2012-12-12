---
layout: default
title: Gosha Arinich
---

<ol class="content articles">
  {% for post in site.posts %}
    <li>
      <article>
        <header>
          <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
          <p class="date">{{ post.date | date_to_string }}</p>
        </header>
      </article>
    </li>
  {% endfor %}
</ol>
