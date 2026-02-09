---
layout: page
title: Browse Foods
---

{% assign sorted_foods = site.foods | sort: "title" %}

{% for food in sorted_foods %}
1. [{{ food.title }}]({{ site.baseurl }}{{ food.url }}) — *{{ food.continent }}* ({{ food.substrate_category }}, {{ food.fermentation_type }})
{% endfor %}

