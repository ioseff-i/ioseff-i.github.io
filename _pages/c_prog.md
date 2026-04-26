---
title: "Introduction to C Programming"
layout: single
permalink: /blog/c_prog/
author_profile: true
---

## Posts

{% assign lag_posts = site.blog | where_exp: "item", "item.path contains 'c_prog'" %}
{% for post in lag_posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}