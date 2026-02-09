---
layout: page
title: World Fermented Foods Map
---

<style>
  .map-toolbar {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    align-items: flex-end;
    margin: 10px 0 14px 0;
    padding: 12px;
    border: 1px solid rgba(0,0,0,0.12);
    border-radius: 10px;
    background: rgba(255,255,255,0.85);
    backdrop-filter: blur(6px);
  }
  .map-toolbar .group { display: flex; flex-direction: column; gap: 6px; min-width: 220px; }
  .map-toolbar label { font-size: 12px; opacity: 0.8; }
  .map-toolbar select, .map-toolbar button {
    padding: 8px 10px;
    border-radius: 8px;
    border: 1px solid rgba(0,0,0,0.18);
    background: white;
    font-size: 14px;
  }
  .map-toolbar button { cursor: pointer; }
  .stats {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    margin: 10px 0 14px 0;
  }
  .stat-card {
    border: 1px solid rgba(0,0,0,0.12);
    border-radius: 10px;
    padding: 10px 12px;
    background: white;
    min-width: 220px;
  }
  .stat-card .big { font-size: 22px; font-weight: 700; }
  .stat-card .small { font-size: 12px; opacity: 0.8; }
  .legend {
    background: white;
    padding: 10px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  }

  /* Pro cluster styling */
  .marker-cluster-custom {
    border-radius: 999px;
    color: white;
    font-weight: 700;
    text-align: center;
    border: 2px solid rgba(255,255,255,0.9);
    box-shadow: 0 3px 10px rgba(0,0,0,0.25);
    user-select: none;
  }
</style>

<div class="map-toolbar">
  <div class="group">
    <label for="filterContinent">Continent</label>
    <select id="filterContinent"></select>
  </div>
  <div class="group">
    <label for="filterSubstrate">Substrate category</label>
    <select id="filterSubstrate"></select>
  </div>
  <div class="group">
    <label for="filterFermentation">Fermentation type</label>
    <select id="filterFermentation"></select>
  </div>
  <div class="group" style="min-width:140px;">
    <label>&nbsp;</label>
    <button id="resetBtn">Reset</button>
  </div>
</div>

<div class="stats">
  <div class="stat-card">
    <div class="small">Foods plotted (with coordinates)</div>
    <div class="big" id="statTotal">0</div>
  </div>
  <div class="stat-card">
    <div class="small">By continent (visible)</div>
    <div id="statByContinent" style="font-size:14px; line-height:1.5;"></div>
  </div>
</div>

<div id="map" style="height: 650px; width: 100%; border-radius: 12px;"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css"/>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<script>
  // ---------- Data from your _foods collection ----------
  const FOODS = [
    {% assign sorted_foods = site.foods | sort: "title" %}
    {% for food in sorted_foods %}
      {% if food.lat and food.lon %}
      {
        title: {{ food.title | jsonify }},
        continent: {{ food.continent | default: "Other" | jsonify }},
        substrate: {{ food.substrate_category | default: "Unknown" | jsonify }},
        fermentation: {{ food.fermentation_type | default: "Unknown" | jsonify }},
        lat: {{ food.lat }},
        lon: {{ food.lon }},
        url: {{ site.baseurl | append: food.url | jsonify }}
      }{% unless forloop.last %},{% endunless %}
      {% endif %}
    {% endfor %}
  ];

  // ---------- Map setup ----------
  const map = L.map('map', { worldCopyJump: true }).setView([20, 0], 2);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  function colorForContinent(c) {
    c = (c || "").toLowerCase();
    if (c.includes("africa")) return "#2ca02c";
    if (c.includes("asia")) return "#d62728";
    if (c.includes("europe")) return "#1f77b4";
    if (c.includes("americas")) return "#ff7f0e";
    if (c.includes("oceania")) return "#9467bd";
    return "#7f7f7f";
  }

  function escapeHtml(s) {
    return String(s ?? "")
      .replaceAll("&", "&amp;")
      .replaceAll("<", "&lt;")
      .replaceAll(">", "&gt;")
      .replaceAll('"', "&quot;")
      .replaceAll("'", "&#039;");
  }

  // ---------- Pro cluster group ----------
  const clusters = L.markerClusterGroup({
    showCoverageOnHover: false,
    spiderfyOnMaxZoom: true,
    disableClusteringAtZoom: 7,

    iconCreateFunction: function (cluster) {
      const children = cluster.getAllChildMarkers();

      // Count markers by continent
      const counts = {};
      for (const m of children) {
        const c = (m.options.continent || "Other");
        counts[c] = (counts[c] || 0) + 1;
      }

      // Majority continent determines cluster color
      let major = "Other";
      let max = -1;
      for (const [k, v] of Object.entries(counts)) {
        if (v > max) { max = v; major = k; }
      }
      const col = colorForContinent(major);

      // Tooltip breakdown
      const breakdown = Object.entries(counts)
        .sort((a,b) => b[1] - a[1])
        .map(([k,v]) => `${k}: ${v}`)
        .join(" | ");

      // Cluster size styling
      const n = children.length;
      let size = 34;
      if (n >= 20) size = 46;
      else if (n >= 10) size = 40;

      const html = `
        <div class="marker-cluster-custom"
             title="${breakdown.replace(/"/g, '&quot;')}"
             style="background:${col}; width:${size}px; height:${size}px; line-height:${size}px;">
          ${n}
        </div>`;

      return L.divIcon({
        html,
        className: "",        // use our own HTML class
        iconSize: [size, size]
      });
    }
  });

  function makeMarker(food) {
    const col = colorForContinent(food.continent);

    return L.circleMarker([food.lat, food.lon], {
      radius: 5,
      weight: 1,
      opacity: 1,
      fillOpacity: 0.85,
      color: col,
      fillColor: col,

      // critical: used for cluster majority + breakdown
      continent: food.continent
    }).bindPopup(
      `<b>${escapeHtml(food.title)}</b><br>` +
      `${escapeHtml(food.continent)}<br>` +
      `${escapeHtml(food.substrate)} — ${escapeHtml(food.fermentation)}<br>` +
      `<a href="${food.url}">View entry</a>`
    );
  }

  // ---------- Filters UI ----------
  const elCont = document.getElementById("filterContinent");
  const elSub  = document.getElementById("filterSubstrate");
  const elFer  = document.getElementById("filterFermentation");
  const elReset = document.getElementById("resetBtn");

  function uniqSorted(values) {
    return Array.from(new Set(values.map(v => (v || "Other").trim())))
      .sort((a,b) => a.localeCompare(b));
  }

  function fillSelect(el, options, labelAll) {
    el.innerHTML = "";
    const optAll = document.createElement("option");
    optAll.value = "";
    optAll.textContent = labelAll;
    el.appendChild(optAll);
    for (const v of options) {
      const o = document.createElement("option");
      o.value = v;
      o.textContent = v;
      el.appendChild(o);
    }
  }

  fillSelect(elCont, uniqSorted(FOODS.map(f => f.continent)), "All continents");
  fillSelect(elSub,  uniqSorted(FOODS.map(f => f.substrate)), "All substrates");
  fillSelect(elFer,  uniqSorted(FOODS.map(f => f.fermentation)), "All fermentation types");

  // ---------- Render + stats ----------
  function render() {
    const c = elCont.value;
    const s = elSub.value;
    const f = elFer.value;

    const visible = FOODS.filter(x =>
      (!c || x.continent === c) &&
      (!s || x.substrate === s) &&
      (!f || x.fermentation === f)
    );

    clusters.clearLayers();
    for (const item of visible) clusters.addLayer(makeMarker(item));
    if (!map.hasLayer(clusters)) map.addLayer(clusters);

    updateStats(visible);
  }

  function updateStats(list) {
    document.getElementById("statTotal").textContent = list.length.toString();

    const counts = {};
    for (const x of list) counts[x.continent] = (counts[x.continent] || 0) + 1;

    const lines = Object.entries(counts)
      .sort((a,b) => b[1] - a[1])
      .map(([k,v]) => `<span style="color:${colorForContinent(k)}">●</span> ${escapeHtml(k)}: <b>${v}</b>`)
      .join("<br>");

    document.getElementById("statByContinent").innerHTML =
      lines || "<span style='opacity:0.8;font-size:12px;'>No points match the current filters.</span>";
  }

  elCont.addEventListener("change", render);
  elSub.addEventListener("change", render);
  elFer.addEventListener("change", render);

  elReset.addEventListener("click", () => {
    elCont.value = "";
    elSub.value = "";
    elFer.value = "";
    render();
  });

  // Initial render
  render();

  // ---------- Legend ----------
  const legend = L.control({ position: 'bottomright' });
  legend.onAdd = function () {
    const div = L.DomUtil.create('div', 'legend');
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

<p style="margin-top:10px; opacity:0.8;">
  Note: only foods with <code>lat</code> and <code>lon</code> appear on the map. You can fill missing coordinates later.
</p>
