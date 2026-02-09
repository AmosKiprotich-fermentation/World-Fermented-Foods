---
layout: page
title: World Fermented Foods Map
---

<div id="map" style="height: 650px; width: 100%;"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
  var map = L.map('map').setView([20, 0], 2);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  var count = 0;

  {% for food in site.foods %}
    {% if food.lat and food.lon %}
      count++;
      L.marker([{{ food.lat }}, {{ food.lon }}])
        .addTo(map)
        .bindPopup(
          "<b>{{ food.title | escape }}</b><br>" +
          "{{ food.continent | escape }}<br>" +
          "{{ food.substrate_category | escape }} — {{ food.fermentation_type | escape }}<br>" +
          "<a href='{{ site.baseurl }}{{ food.url }}'>View entry</a>"
        );
    {% endif %}
  {% endfor %}

  console.log("Plotted markers:", count);
</script>

<p><em>Tip:</em> If some foods don’t appear, they likely don’t have <code>lat</code> and <code>lon</code> in their front matter yet.</p>
