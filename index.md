---
layout: page
title: Beyond All Regognition
tagline: lubar is fubar
---
<ul class="posts">
  {% for post in site.posts %}
    <li>
      <p class="post-preview">
        <div class="post-preview-image">
          <a href="{{ BASE_PATH }}{{ post.url }}" class="img-link">
            <img src="{{ post.image }}" height="200" />
          </a>
          <div class="post-preview-title">
            <a href="{{ BASE_PATH }}{{ post.url }}">
              {{ post.title }}
              <span class="post-preview-date">{{ post.date | date_to_string }}</span>
            </a>
          </div>
        </div>
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

