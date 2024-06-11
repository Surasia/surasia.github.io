---
layout: default
title: Posts
permalink: /blog
pagination:
    enabled: true
---

<h2>Posts</h2>
{% for post in site.posts %}
<span class="post-item"><span class="mobile-hide"> {{ post.date | date: '%Y-%m-%d' }} >> </span><a href="{{ post.url }}">{{ post.title }}</a></span>
{% endfor %}