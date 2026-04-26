---
title: "Explanations of Some Research Papers"
layout: single
permalink: /blog/paper-templates/
author_profile: true
---

## Posts

{% assign lag_posts = site.blog | where_exp: "item", "item.path contains 'paper-templates'" %}
{% for post in lag_posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}