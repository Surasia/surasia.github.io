---
layout: default
title: Home
---
Welcome to my blog! This site covers some research and reverse engineering
projects that I'm working on. This ranges from decompilation and malware
analysis to game modding.

<br>
Social Links:
- [Github](https://github.com/Surasia)
- [Twitter](https://x.com/Surasia_)
- [Youtube](https://www.youtube.com/@Surasia)

<br>
Recent posts:
{% for post in site.posts %}
- <span class="post-item"><span class="mobile-hide"> {{ post.date | date: '%d/%m/%Y' }} >> </span><a href="{{ post.url }}">{{ post.title }}</a></span>
{% endfor %}