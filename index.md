---
layout: page
title: Beyond All Regognition
tagline: lubar is fubar
---
Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <span>{{ post.date | date_to_string }}</span>
      &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
      <p>{{ post.excerpt }}</p>
      </li>
  {% endfor %}
</ul>

