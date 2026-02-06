---
layout: page
title: Map
---

<div id="map" style="height: 650px; width: 100%; border-radius: 12px;"></div>

<link
  rel="stylesheet"
  href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
  // Create map
  const map = L.map("map").setView([15, 0], 2);

  // Add basemap tiles
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 18,
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  // Load your foods data
  fetch("{{ site.baseurl }}/assets/data/foods.json")
    .then(res => res.json())
    .then(data => {
      data.forEach(item => {
        const popup = `
          <b>${item.name}</b><br/>
          ${item.country} — ${item.continent}<br/>
          ${item.substrate}, ${item.fermentation_type}<br/>
          <a href="${item.url}">Open entry</a>
        `;

        L.marker([item.lat, item.lon]).addTo(map).bindPopup(popup);
      });
    })
    .catch(err => console.error("Failed to load foods.json:", err));
</script>
