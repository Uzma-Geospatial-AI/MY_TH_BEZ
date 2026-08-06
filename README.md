# MY–TH BEZ — Border Economic Zone GIS

An interactive, single-page GIS dashboard for the **Malaysia–Thailand Border
Economic Zone** under the Northern Corridor Economic Region (NCER), built by
**Geospatial AI Sdn Bhd**, an Uzma Group company. The interface is themed with a
batik-inspired palette (`#B87800` gold · `#E8450A` terracotta · `#0F8C72` teal)
on a Playfair Display / Noto Sans type pairing.

Live site: https://uzma-geospatial-ai.github.io/MY_TH_BEZ/

The dashboard ships six modes:

1. **Investor Dashboard** — industrial zones, border nodes and prime investment opportunities across the corridor
2. **Tourism Guide** — UNESCO sites, National Geoparks, heritage villages, duty-free zones and coastal destinations
3. **CVIA Site Select** — Chuping Valley Industrial Area, 2,482 ha across six sub-zones with parcel, suitability and infrastructure views
4. **Kedah Rubber City** — Malaysia's first dedicated rubber eco-industrial park, 1,244 acres in Padang Terap
5. **Layer Control** — category toggles, feature filters and basemap switching
6. **Border Movement Analytics** — 8 MY–TH checkpoints, 2018–2025 arrivals and departures by nationality

Designed as an **enterprise GIS dashboard** — a fixed command bar, a collapsible
analysis panel and a full-bleed Leaflet canvas — with no build step and no
framework runtime. Access is gated behind a sign-in page.

---

## Tech stack

- **Vanilla HTML + CSS + JavaScript** — no bundler, no framework, no `node_modules`
- **Leaflet 1.9.4** — map engine, custom panes for explicit z-ordering
- **Chart.js 4.4.1** — donut, line, bar and comparison charts in the analysis panel
- **GeoJSON** — rail, boundary and zone geometry exported from QGIS
- **Google Fonts** — Playfair Display (display) + Noto Sans (UI)
- **Basemaps** — OpenStreetMap · Esri World Imagery · CARTO Dark · OpenTopoMap

Everything except the two CDN libraries, the fonts and the basemap tiles is
served from this repository.

---

## Folder structure

```
MY_TH_BEZ/
├── index.html               # dashboard — markup, styles, map & chart logic (587 KB)
├── login.html               # access gate / sign-in page (28 KB)
├── geodata.js               # rail, border & BEZ GeoJSON (367 KB)
├── pergerakan.js            # border-crossing movement data 2018–2025 (20 KB)
├── geospatial-ai-logo.png   # brand mark, transparent (874×193)
└── README.md
```

`index.html` is deliberately self-contained: the `:root` design tokens, every
component style, the Leaflet setup and all six panel renderers live in one file
so the dashboard can be dropped onto any static host as-is.

---

## Local development

There is nothing to install and nothing to build.

```bash
# 1. serve the folder over HTTP
python -m http.server 8000
# or
npx serve .

# 2. open
# → http://localhost:8000/login.html
```

> Opening `index.html` straight off disk (`file://`) also works in Chrome, but a
> local HTTP server is closer to how GitHub Pages serves it. An internet
> connection is required either way — Leaflet, Chart.js, the fonts and the
> basemap tiles all come from CDNs.

---

## Modes

| # | Mode                | Topbar button | Pane id           | What it renders                                              |
|---|---------------------|---------------|-------------------|--------------------------------------------------------------|
| 1 | Investor Dashboard  | Investor      | `pane-investor`   | KPI cards, feature-distribution donut, visitor trend, zones by region |
| 2 | Tourism Guide       | Tourism       | `pane-tourist`    | Nature, tourism and shopping categories with site cards      |
| 3 | CVIA Site Select    | CVIA          | `pane-cvia`       | Overview · Parcels · Suitability · Infra tabs, six sub-zones |
| 4 | Kedah Rubber City   | KRC           | `pane-krc`        | Park profile, investment targets, NCER reference link        |
| 5 | Layer Control       | Layers        | `pane-layers`     | Category toggles, vector-layer switches, basemap picker      |
| 6 | Movement Analytics  | Analytics     | `pane-analytics`  | Nationality + year filters, annual trend, monthly pattern, summary table |

`setMode('<id>')` swaps the active pane and re-syncs the map layers.

---

## Map data

**116 point features** across nine categories, defined in `index.html` as `F[]`
and coloured from the `CATS` lookup:

| Category           | Key               | Count | Colour    |
|--------------------|-------------------|-------|-----------|
| Industrial Parks   | `industrial`      | 25    | `#7E57C2` |
| Universities / HLI | `university`      | 18    | `#43A047` |
| Border Crossings   | `border_crossing` | 17    | `#E53935` |
| Duty Free / Shopping | `shopping`      | 13    | `#C9910E` |
| Tourism Sites      | `tourism`         | 11    | `#C4622D` |
| Ports & Jetties    | `port`            | 10    | `#00ACC1` |
| Railway / ECRL     | `railway`         | 8     | `#FB8C00` |
| Nature & Geoparks  | `nature`          | 8     | `#558B2F` |
| Airports           | `airport`         | 6     | `#1E88E5` |

Vector layers from `geodata.js`:

| Constant                | Geometry        | Layer                          |
|-------------------------|-----------------|--------------------------------|
| `ECRL_DATA`             | MultiLineString | East Coast Rail Link           |
| `KTMB_DATA`             | MultiLineString | KTMB operational rail          |
| `KTMB_NON_DATA`         | MultiLineString | KTMB non-operational rail      |
| `MY_BEZ_DATA`           | Polygon         | Malaysian BEZ districts        |
| `TH_BEZ_DATA`           | Polygon         | Thai BEZ provinces             |
| `MY_TH_BOUNDARY_DATA`   | MultiLineString | International boundary         |

Plus `INDPARK_GEOJSON` (industrial park polygons, QGIS export) and
`CVIA_BOUNDARY` / `CVIA_ZONES` (six Chuping Valley sub-zones) in `index.html`.

`pergerakan.js` holds `CROSSINGS_DATA` for eight checkpoints — Bukit Kayu Hitam,
ICQ Durian Burung, Padang Besar, Pengkalan Hulu, Pengkalan Kubur, Pos Imm Bukit
Bunga, Rantau Panjang and Wang Kelian — with yearly and monthly arrivals and
departures split by Malaysian, Thai and other nationalities.

---

## Access

`login.html` is the entry point. `index.html` carries a gate script at the top of
`<head>` that redirects unauthenticated visitors before anything renders.

- **No sign-up path** — a single credential set, issued by Geospatial AI Support
- The password is never stored in source; the page keeps only a **salted,
  4,096-round SHA-256 digest**, split across reversed chunks and reassembled at
  runtime
- SHA-256 is implemented in-page rather than via `crypto.subtle`, so the gate
  also works over `file://`
- Five failed attempts trigger a 30-second lockout
- On success a session token is written to `localStorage` (30 days, *Remember
  me*) or `sessionStorage` (12 hours); **Sign Out** in the topbar clears both

> This is client-side gating. It keeps casual visitors out of the dashboard, but
> the repository is public, so treat the source itself as public. Anything that
> genuinely must stay private belongs behind a server.

---

## Theming

Design tokens are declared once in `:root` and reused across both pages.

| Token          | Value     | Role                                        |
|----------------|-----------|---------------------------------------------|
| `--gold`       | `#B87800` | Borders, rules, wordmark shimmer            |
| `--terr`       | `#E8450A` | Primary accent — CTAs, active states        |
| `--terr2`      | `#F26A1B` | Accent highlight, micro-labels              |
| `--teal2`      | `#0F8C72` | Live indicators, secondary metrics          |
| `--sage`       | `#1E7A3A` | Positive / supporting metrics               |
| `--java`       | `#F0F7F4` | Page background                             |
| `--text`       | `#1A2E1A` | Body text                                   |

A batik SVG motif is tiled at 4.5% opacity over both the dashboard and the login
page, which is what ties the two together visually.

---

## Deployment

The repository is served directly by **GitHub Pages** from `main` — there is no
build artefact and no `dist/`.

```bash
git add .
git commit -m "your change"
git push origin main
# Pages rebuilds in ~40s
```

Any other static host works the same way — upload the repository contents as-is:

```bash
# S3 / Cloudflare Pages / Netlify drop
# no build command, publish directory = repository root
```

`index.html` is the default document, so a visitor landing on the site root is
redirected to `login.html` by the gate script.

---

## Credits

- Border movement data — **JIM / NCIA**, `Data_Pergerakan_NCER_2018-2025.xlsx`
- CVIA and KRC zone profiles — [NCER](https://www.ncer.com.my/)
- Basemap tiles — [OpenStreetMap](https://www.openstreetmap.org/), [Esri World Imagery](https://www.arcgis.com/), [CARTO](https://carto.com/), [OpenTopoMap](https://opentopomap.org/)
- Map engine — [Leaflet](https://leafletjs.com/)
- Charts — [Chart.js](https://www.chartjs.org/)
- Feature card imagery — [Unsplash](https://unsplash.com/)
- Type — [Playfair Display](https://fonts.google.com/specimen/Playfair+Display) + [Noto Sans](https://fonts.google.com/noto/specimen/Noto+Sans)
