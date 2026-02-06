---
layout: page
title: Fermented Foods
---

## Global Fermented Foods Index

{% assign sorted_foods = site.foods | sort: "title" %}

{% for food in sorted_foods %}
- [{{ food.title }}]({{ site.baseurl }}{{ food.url }}) — *{{ food.continent }}* ({{ food.substrate_category }}, {{ food.fermentation_type }})
{% endfor %}
