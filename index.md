---
layout: page
title: Beyond All Regognition
tagline: lubar is fubar
---
<ul class="posts">
  {% for post in site.posts %}
    <li class="post-preview">
      <p class="post-preview" style="background-image:url('{{ post.image }}');">
        <a href="{{ BASE_PATH }}{{ post.url }}" class="post-preview-title">
            {{ post.title }}
            <span class="post-preview-date">{{ post.date | date_to_string }}</span>
        </a>
      </p>
      <p>
        {{ post.excerpt }}
        <a href="{{ BASE_PATH}}{{ post.url }}" class="post-preview-read-more">
          <i>&raquo; Read More… </i>
        </a>
      </p>
    </li>
  {% endfor %}
</ul>

