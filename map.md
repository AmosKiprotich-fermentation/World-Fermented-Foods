---
layout: page
title: Explore Map
---

<style>
  :root{
    --border: rgba(0,0,0,0.12);
    --border2: rgba(0,0,0,0.18);
    --shadow: 0 2px 10px rgba(0,0,0,0.10);
    --shadow2: 0 6px 18px rgba(0,0,0,0.14);
    --radius: 12px;
  }

  .map-intro{
    max-width: 980px;
    margin: 6px 0 12px 0;
    opacity: 0.88;
    line-height: 1.45;
  }

  .map-grid{
    display: grid;
    grid-template-columns: 1fr;
    gap: 12px;
    margin: 10px 0 14px 0;
  }

  .panel{
    border: 1px solid var(--border);
    border-radius: var(--radius);
    background: #fff;
    box-shadow: var(--shadow);
    padding: 12px;
  }

  .map-toolbar{
    display: grid;
    grid-template-columns: repeat(4, minmax(180px, 1fr));
    gap: 10px;
    align-items: end;
  }
  @media (max-width: 900px){
    .map-toolbar{ grid-template-columns: repeat(2, minmax(180px, 1fr)); }
  }
  @media (max-width: 520px){
    .map-toolbar{ grid-template-columns: 1fr; }
  }

  .group{ display:flex; flex-direction:column; gap:6px; }
  label{ font-size: 12px; opacity: 0.82; }

  select, button{
    padding: 9px 10px;
    border-radius: 10px;
    border: 1px solid var(--border2);
    background: #fff;
    font-size: 14px;
    line-height: 1.2;
  }
  button{ cursor: pointer; }
  button:hover{ box-shadow: 0 1px 6px rgba(0,0,0,0.10); }
  select:focus, button:focus{
    outline: 2px solid rgba(31,119,180,0.35);
    outline-offset: 2px;
  }

  .stats{
    display: grid;
    grid-template-columns: 260px 1fr;
    gap: 12px;
  }
  @media (max-width: 900px){
    .stats{ grid-template-columns: 1fr; }
  }

  .stat-card .big{ font-size: 24px; font-weight: 750; letter-spacing: -0.02em; }
  .stat-card .small{ font-size: 12px; opacity: 0.78; margin-bottom: 6px; }

  .legend{
    background: #fff;
    padding: 10px;
    border-radius: 12px;
    box-shadow: var(--shadow2);
    border: 1px solid var(--border);
  }

  /* Pro cluster styling */
  .marker-cluster-custom{
    border-radius: 999px;
    color: #fff;
    font-weight: 750;
    text-align: center;
    border: 2px solid rgba(255,255,255,0.95);
    box-shadow: 0 3px 12px rgba(0,0,0,0.22);
    user-select: none;
  }

  /* nicer Leaflet popup typography */
  .leaflet-popup-content{
    margin: 10px 12px;
    line-height: 1.35;
  }
  .popup-title{
    font-weight: 800;
    letter-spacing: -0.01em;
    margin-bottom: 4px;
  }
  .popup-meta{
    opacity: 0.85;
    font-size: 13px;
    margin-bottom: 8px;
  }
  .popup-link a{
    text-decoration: none;
    font-weight: 650;
  }

  .note{
    font-size: 12px;
    opacity: 0.72;
    margin-top: 6px;
  }
</style>

<p class="map-intro">
An open, curated database of traditional fermented foods, organised by substrate, fermentation type, and geographic origin.
Use the filters to explore patterns across regions and fermentation systems.
</p>

<div class="map-grid">
  <div class="panel">
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
      <div class="group">
        <label>&nbsp;</label>
        <button id="resetBtn" aria-label="Reset filters">Reset</button>
      </div>
    </div>
  </div>

  <div class="stats">
    <div class="panel stat-card">
      <div class="small">Foods plotted (with coordinates)</div>
      <div class="big" id="statTotal">0</div>
      <div class="note">Filtered totals update live.</div>
    </div>

    <div class="panel stat-card">
      <div class="small">By continent (visible)</div>
      <div id="statByContinent" style="font-size:14px; line-height:1.55;"></div>
    </div>
  </div>

  <div id="map" style="height: 650px; width: 100%; border-radius: 14px; border: 1px solid var(--border); box-shadow: var(--shadow2);"></div>

  <div class="note">
    Note: only foods with <code>lat</code> and <code>lon</code> appear on the map. You can fill missing coordinates later.
  </div>
</div>

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

  // ---------- Pro cluster group (majority continent color + breakdown tooltip) ----------
  const clusters = L.markerClusterGroup({
    showCoverageOnHover: false,
    spiderfyOnMaxZoom: true,
    disableClusteringAtZoom: 7,

    iconCreateFunction: function (cluster) {
      const children = cluster.getAllChildMarkers();

      const counts = {};
      for (const m of children) {
        const c = (m.options.continent || "Other");
        counts[c] = (counts[c] || 0) + 1;
      }

      let major = "Other";
      let max = -1;
      for (const [k, v] of Object.entries(counts)) {
        if (v > max) { max = v; major = k; }
      }
      const col = colorForContinent(major);

      const breakdown = Object.entries(counts)
        .sort((a,b) => b[1] - a[1])
        .map(([k,v]) => `${k}: ${v}`)
        .join(" | ");

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
        className: "",
        iconSize: [size, size]
      });
    }
  });

  function popupHtml(food){
    return `
      <div class="popup-title">${escapeHtml(food.title)}</div>
      <div class="popup-meta">${escapeHtml(food.continent)} • ${escapeHtml(food.substrate)} • ${escapeHtml(food.fermentation)}</div>
      <div class="popup-link"><a href="${food.url}">Open entry →</a></div>
    `;
  }

  function makeMarker(food) {
    const col = colorForContinent(food.continent);

    return L.circleMarker([food.lat, food.lon], {
      radius: 4,          // slightly smaller = cleaner
      weight: 1,
      opacity: 1,
      fillOpacity: 0.80,
      color: col,
      fillColor: col,
      continent: food.continent
    }).bindPopup(popupHtml(food));
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
      lines || "<span style='opacity:0.75;font-size:12px;'>No points match the current filters.</span>";
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
