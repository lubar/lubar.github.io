---
layout: page
tagline: lubar is fubar
---
<ul class="posts">
  {% for post in site.posts limit:1 %}
    <li>
      <div class="post-preview">
        <a href="{{ BASE_PATH }}{{ post.url }}" class="post-preview-title" style="position:absolute;">
          {{ post.title }}
          <span class="post-preview-date">{{ post.date | date_to_string }}</span>
        </a>
      </div>
      <p>
        {{ post.content | remove: '<p>' | remove: '</p>' }}
      </p>
    </li>
  {% endfor %}

  {% for post in site.posts offset:1 limit:9 %}
    <li>
		{% if post.image != null %}
		<a href="{{ BASE_PATH }}{{ post.url }}" class="img-link">
          <img src="{{ post.image }}" height="280" style="margin:0;"/>
        </a>
		{% endif %}
		<div class="post-preview">
			<a href="{{ BASE_PATH }}{{ post.url }}" class="post-preview-title">
				{{ post.title }}
				<span class="post-preview-date">{{ post.date | date_to_string }}</span>
			</a>
		</div>
		<p>
			{{ post.excerpt | remove: '<p>' | remove: '</p>' }}
			<a href="{{ BASE_PATH}}{{ post.url }}" class="post-preview-read-more">
				<i>&raquo; Read More… </i>
			</a>
		</p>
    </li>
  {% endfor %}
</ul>

