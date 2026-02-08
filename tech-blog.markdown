---
layout: page
title: Tech Blog
permalink: /tech-blog/
---

Welcome to my tech blog where I share technical insights, programming tutorials, and development experiences.

<ul class="post-list">
  {%- for post in site.posts -%}
    {%- if post.categories contains 'tech' -%}
      <li class="post-list-item">
        <h3 class="post-title">
          <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
        </h3>
        <div class="post-meta">
          <time datetime="{{ post.date | date_to_xmlschema }}">
            {{ post.date | date: site.date_format | default: "%b %-d, %Y" }}
          </time>
          {%- if post.categories.size > 0 -%}
            • {{ post.categories | join: ", " }}
          {%- endif -%}
        </div>
        {%- if post.excerpt -%}
          <div class="post-excerpt">
            {{ post.excerpt | strip_html | truncatewords: 30 }}
            <a href="{{ post.url | relative_url }}">Read more →</a>
          </div>
        {%- endif -%}
      </li>
    {%- endif -%}
  {%- endfor -%}
</ul>

{%- if site.posts == empty or site.posts.size == 0 or site.posts contains 'tech' == false -%}
  <p><em>No tech posts yet. Check back soon!</em></p>
{%- endif -%}