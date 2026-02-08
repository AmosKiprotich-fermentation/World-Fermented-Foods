---
layout: page
title: World Fermented Foods Map
---

<div id="map" style="height: 600px; width: 100%;"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
  var map = L.map('map').setView([0, 0], 2);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  fetch('{{ site.baseurl }}/assets/data/foods.json')
    .then(r => r.json())
    .then(data => {
      data.forEach(item => {
        L.marker([item.lat, item.lon])
          .addTo(map)
          .bindPopup('<b>' + item.name + '</b><br><a href="' + item.url + '">View entry</a>');
      });
    });
</script>
