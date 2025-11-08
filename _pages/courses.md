---
title: "Courses"
layout: single
permalink: /courses/
---

This page lists all available courses.

{% assign top_courses = site.courses | where: "has_children", true %}

<ul>
  {% for course in top_courses %}
    <li>
      <a href="{{ course.url | relative_url }}">{{ course.title }}</a><br>
      {{ course.excerpt }}
    </li>
  {% endfor %}
</ul>
