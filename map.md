---
layout: page
title: World Fermented Foods Map
---

<div id="map" style="height: 650px; width: 100%;"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<!-- Marker clustering -->
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css"/>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<script>
  // Base map
  var map = L.map('map', { worldCopyJump: true }).setView([20, 0], 2);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  // Continent color coding (small dots)
  function colorForContinent(c) {
    c = (c || "").toLowerCase();
    if (c.includes("africa")) return "#2ca02c";
    if (c.includes("asia")) return "#d62728";
    if (c.includes("europe")) return "#1f77b4";
    if (c.includes("americas")) return "#ff7f0e";
    if (c.includes("oceania")) return "#9467bd";
    return "#7f7f7f";
  }

  // Cluster group (reduces congestion)
  var clusters = L.markerClusterGroup({
    showCoverageOnHover: false,
    spiderfyOnMaxZoom: true,
    disableClusteringAtZoom: 7
  });

  // Add markers from your _foods collection
  {% for food in site.foods %}
    {% if food.lat and food.lon %}
      (function() {
        var continent = {{ food.continent | jsonify }};
        var marker = L.circleMarker(
          [{{ food.lat }}, {{ food.lon }}],
          {
            radius: 5,              // small dot size
            weight: 1,              // thin outline
            opacity: 1,
            fillOpacity: 0.85,
            color: colorForContinent(continent),
            fillColor: colorForContinent(continent)
          }
        ).bindPopup(
          "<b>{{ food.title | escape }}</b><br>" +
          "{{ food.continent | escape }}<br>" +
          "{{ food.substrate_category | escape }} — {{ food.fermentation_type | escape }}<br>" +
          "<a href='{{ site.baseurl }}{{ food.url }}'>View entry</a>"
        );

        clusters.addLayer(marker);
      })();
    {% endif %}
  {% endfor %}

  map.addLayer(clusters);

  // Simple legend
  var legend = L.control({ position: 'bottomright' });
  legend.onAdd = function () {
    var div = L.DomUtil.create('div', 'info legend');
    div.style.background = 'white';
    div.style.padding = '10px';
    div.style.borderRadius = '8px';
    div.style.boxShadow = '0 2px 8px rgba(0,0,0,0.15)';
    div.innerHTML =
      "<b>Continent</b><br>" +
      "<span style='color:#2ca02c'>●</span> Africa<br>" +
      "<span style='color:#d62728'>●</span> Asia<br>" +
      "<span style='color:#1f77b4'>●</span> Europe<br>" +
      "<span style='color:#ff7f0e'>●</span> Americas<br>" +
      "<span style='color:#9467bd'>●</span> Oceania<br>" +
      "<span style='color:#7f7f7f'>●</span> Other";
    return div;
  };
  legend.addTo(map);
</script>
