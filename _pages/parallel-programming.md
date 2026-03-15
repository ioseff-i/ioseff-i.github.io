---
title: "Introduction to Parallel Programming"
layout: single
permalink: /blog/parallel-programming/
author_profile: true
---

## Posts

{% assign lag_posts = site.blog | where_exp: "item", "item.path contains 'parallel-programming'" %}
{% for post in lag_posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}