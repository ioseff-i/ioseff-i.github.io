---
title: "Introduction to Programming 1"
layout: single
collection: courses
has_children: true
excerpt: "Learn the basics of Python programming."
---

Welcome to the **Introduction to Python** course!  
Here you’ll learn Python basics.


## Lessons

{% assign lessons = site.courses | where: "parent", "Introduction to Python" %}
<ul>
  {% for lesson in lessons %}
    <li>
      <a href="{{ lesson.url | relative_url }}">{{ lesson.title }}</a><br>
      {{ lesson.excerpt }}
    </li>
  {% endfor %}
</ul>
