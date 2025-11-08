---
title: "Discrete Mathematics"
layout: single
collection: courses
has_children: true
excerpt: "Basics of Discrete Math"
---

Welcome to the **Introduction to Python** course!  
Here you’ll learn Python basics.


## Lessons

{% assign lessons = site.courses | where: "parent", "Basics of Discrete Math" %}
<ul>
  {% for lesson in lessons %}
    <li>
      <a href="{{ lesson.url | relative_url }}">{{ lesson.title }}</a><br>
      {{ lesson.excerpt }}
    </li>
  {% endfor %}
</ul>
