---
layout: home
title: Home
nav_order: 1
#nav_exclude: true
permalink: /:path/
seo:
  type: Course
  name: CS 631 Systems Foundations Spring 2024
---

# CS 631 Systems Foundations
Spring 2024 &nbsp; &nbsp; &nbsp; &nbsp; [Sec 01 Zoom](https://usfca.zoom.us/j/85894844618)  &nbsp; &nbsp; [Sec 02 Zoom](https://usfca.zoom.us/j/82219131312)

{% for module in site.modules %}
{{ module }}
{% endfor %}
