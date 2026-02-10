---
layout: page
title: Blog
permalink: /blog/
description: "Explore my blog covering philosophy, theology, history, and personal reflections. Insights on life, learning, and growth."
keywords: "blog, philosophy, theology, history, personal development, reflections, essays"
---

Welcome to my blog where I share thoughts, experiences, and insights on various topics.

{% assign life_posts = site.posts | where_exp: "post", "post.categories contains 'life'" %}

{% if life_posts.size > 0 %}
<ul class="post-list">
  {% for post in life_posts %}
  <li class="post-list-item">
    <h3 class="post-title">
      <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
    </h3>
    <div class="post-meta">
      <time datetime="{{ post.date | date_to_xmlschema }}">
        {{ post.date | date: site.date_format | default: "%b %-d, %Y" }}
      </time>
      {% if post.categories.size > 0 %}
      • {{ post.categories | join: ", " }}
      {% endif %}
    </div>
    {% if post.excerpt %}
    <div class="post-excerpt">
      {{ post.excerpt | strip_html | truncatewords: 30 }}
      <a href="{{ post.url | relative_url }}">Read more →</a>
    </div>
    {% endif %}
  </li>
  {% endfor %}
</ul>
{% else %}

*No posts yet. Check back soon!*

{% endif %}