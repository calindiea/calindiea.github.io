---
layout: page
title: Tech Blog
permalink: /tech-blog/
description: "Technical insights, programming tutorials, and development experiences from a Data Engineer. Learn about GenAI, data pipelines, and software development."
keywords: "tech blog, programming, software development, data engineering, GenAI, tutorials, technical insights, coding"
---

Welcome to my tech blog where I share technical insights, programming tutorials, and development experiences.

{% assign tech_posts = site.posts | where_exp: "post", "post.categories contains 'tech' or post.categories contains 'programming' or post.categories contains 'development'" %}

{% if tech_posts.size > 0 %}
<ul class="post-list">
  {% for post in tech_posts %}
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

*No tech posts yet. Check back soon!*

{% endif %}