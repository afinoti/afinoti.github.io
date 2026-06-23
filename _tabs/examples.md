---
layout: page
icon: fas fa-layer-group
order: 6
---

# Examples

Real-world systems mapped to TOGAF architectural concepts.

{% assign sorted = site.examples | sort: "title" %}
{% for example in sorted %}
- [{{ example.title }}]({{ example.url }})
{% endfor %}
