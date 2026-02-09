---
layout: page
title: Home
---
<img class="hero-img"
     src="{{ '/assets/img/hero.jpg' | relative_url }}"
     alt="World Fermented Foods">

<div class="hero">
  <div class="hero-text">
    <h1>World Fermented Foods</h1>
    <p class="lead">
      An open, curated global database of traditional fermented foods—linking substrates, fermentation types, geography, and associated microbiota.
    </p>

    <div class="cta-row">
      <a class="btn btn-primary" href="{{ '/foods/' | relative_url }}">Browse foods</a>
      <a class="btn btn-ghost" href="{{ '/map/' | relative_url }}">Explore map</a>
      <a class="btn btn-ghost" href="{{ '/citation_policy/' | relative_url }}">Citation policy</a>
    </div>

    <div class="stats-row">
      {% assign total_foods = site.foods | size %}
      {% assign with_coords = site.foods | where_exp: "f", "f.lat and f.lon" | size %}
      <div class="stat">
        <div class="stat-num">{{ total_foods }}</div>
        <div class="stat-label">Total entries</div>
      </div>
      <div class="stat">
        <div class="stat-num">{{ with_coords }}</div>
        <div class="stat-label">Mapped entries</div>
      </div>
      <div class="stat">
        <div class="stat-num">{{ total_foods | minus: with_coords }}</div>
        <div class="stat-label">Missing coordinates</div>
      </div>
    </div>
  </div>
</div>

<div class="card-grid">
  <div class="card">
    <div class="card-title">Structured metadata</div>
    <div class="card-body">Each entry captures continent, substrate category, fermentation type, dominant microbes, and references.</div>
  </div>
  <div class="card">
    <div class="card-title">Research-ready</div>
    <div class="card-body">Designed for comparative microbiology, food science learning, and reproducible citation practices.</div>
  </div>
  <div class="card">
    <div class="card-title">Interactive map</div>
    <div class="card-body">Explore foods geographically and filter by continent, substrate, and fermentation type.</div>
  </div>
</div>

<hr class="soft-rule">

<div class="quick-links">
  <h2>Get started</h2>
  <ul>
    <li><a href="{{ '/foods/' | relative_url }}">Browse the full index of fermented foods</a></li>
    <li><a href="{{ '/map/' | relative_url }}">Explore the interactive world map</a></li>
    <li><a href="{{ '/citation_policy/' | relative_url }}">Read the citation policy</a></li>
  </ul>
</div>
