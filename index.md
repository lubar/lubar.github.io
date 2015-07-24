---
layout: page
tagline: lubar is fubar
---
  {% for post in site.posts limit:1 %}
  <article id="{{post.title}}">
		<div class="post-preview">
        <a href="{{ BASE_PATH }}{{ post.url }}" class="post-preview-title" style="position:absolute;">
          {{ post.title }}
          <span class="post-preview-date">{{ post.date | date_to_string }}</span>
        </a>
      </div>
      <p>
        {{ post.content | remove: '<p>' | remove: '</p>' }}
      </p>
  </article>
  <hr>
  {% endfor %}

  {% for post in site.posts offset:1 limit:9 %}
    <article id="{{post.title}}">
		{% if post.image != null %}
		<a href="{{ BASE_PATH }}{{ post.url }}" class="img-link">
          <img src="{{ post.image }}" height="280" style="margin:0;"/>
        </a>
		{% else %}
		<div style="margin-top:110px">
			&nbsp;
		</div>
		
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
    </article>
  {% endfor %}
