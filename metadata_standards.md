---
layout: page
title: Metadata Standards
---

## Minimum metadata standards

Each food entry is a Markdown file in `_foods/` with YAML front matter. To ensure the database is searchable, comparable, and map-ready, entries should include the following fields.

---

## A) Required fields (must include)

**Front matter (YAML):**
- `title` — canonical food name (Title Case)
- `continent` — one of: `Africa`, `Asia`, `Europe`, `Americas`, `Oceania`, `Antarctica` (Antarctica rarely used)
- `countries` — country or list (e.g., `Japan` or `Japan, Korea`)
- `substrate_category` — broad substrate class (e.g., `Dairy (Milk)`, `Cereal (Maize)`, `Cassava`, `Vegetables`, `Fish`)
- `fermentation_type` — high-level type (e.g., `Lactic acid fermentation (spontaneous)`)
- `dominant_microbes` — major organisms or groups (e.g., `Lactobacillus spp., Saccharomyces spp.`)
- `ref_tags` — comma-separated tags (see Reference Tags)
- `references` — at least 1 credible reference (in the body under “## References”)

**Body sections (Markdown):**
- `## Overview`
- `## Raw Materials`
- `## Fermentation Process`
- `## Microbial Ecology`
- `## Functional and Nutritional Aspects`
- `## Cultural Significance`
- `## References`

---

## B) Recommended fields (strongly encouraged)

**Front matter (YAML):**
- `regions` — region/state/province (e.g., `Oaxaca`, `Bavaria`)
- `local_names` — alternative names
- `starter_used` — `yes/no` (or `starter` vs `spontaneous`)
- `fermentation_time` — e.g., `1–3 days`
- `fermentation_temp` — e.g., `20–30 °C`

---

## C) Map fields (optional but ideal)

To appear on the map, include:
- `lat`
- `lon`

If unknown, leave them out and add later.

---

## D) Controlled vocabulary (to keep data consistent)

**Continents:** `Africa | Asia | Europe | Americas | Oceania | Antarctica`

**Common substrate categories (examples):**
- `Dairy (Milk)`
- `Cereal (Maize)`
- `Cereal (Rice)`
- `Cereal (Wheat/Rye)`
- `Cassava`
- `Vegetables`
- `Fruit`
- `Fish`
- `Meat`
- `Plant sap (Palm/Agave)`
- `Legumes (Soybean)`

---

## E) Quality rule

If a claim is specific (e.g., naming species, reporting ethanol %, linking to health effects), it should be supported by a reference.
