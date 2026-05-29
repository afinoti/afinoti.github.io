---
layout: page
icon: fas fa-book
order: 5
---

# Concepts

Definitions and fundamental concepts in computer science.

{% assign sorted = site.concepts | sort: "title" %}
{% for concept in sorted %}
- [{{ concept.title }}]({{ concept.url }})
{% endfor %}
