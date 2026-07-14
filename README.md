# Datum Point

A single-page coordinate transformation calculator. Pick a source and target coordinate system, pick a datum transformation (ranked best-to-worst for your points' location), enter one point, paste a list, or upload a CSV — get output you can copy or download. No GIS required.

Live math, no server: everything runs in the browser.

## What's inside

```
index.html          the entire app (UI, engine, EPSG catalog — one file)
grids/              63 NOAA grid-shift files in compact binary form (.csg)
grids/manifest.json grid file sizes
README.md
```

## Deploying to GitHub Pages

Every file is under 25 MB (the two large NADCON5 grids ship pre-split into parts the app reassembles automatically), so the whole thing can be deployed from a phone through github.com — no git, no computer. See the step-by-step phone instructions in the project notes, or if you're on a computer with git: create a repo, push the folder, enable Pages from main / (root).

Grids are fetched on demand, so users only download the grid a chosen transformation actually needs (most are under 1 MB).

Note: opening `index.html` directly from disk (file://) works for parameter-based transformations but not grid-based ones, because browsers block local `fetch`. Host it (GitHub Pages, or `python -m http.server` locally) for full function.


## Coverage

- **Datums:** NAD27, NAD83(1986), NAD83(HARN), NAD83(2011), WGS 84
- **Systems:** all US State Plane zones (NAD27, NAD83, NAD83(2011); meters, ftUS, and ft variants), UTM zones for NAD27/NAD83/NAD83(2011)/WGS 84, Web Mercator, Albers national/state systems, and the geographic systems themselves — about 720 total
- **Transformations:** 222 operations from the EPSG Geodetic Parameter Dataset — NADCON, NADCON5, and HARN/HPGN grid shifts plus Helmert and 3-parameter shifts — each carrying EPSG's published accuracy and area of use, which drive the ranking

## Accuracy and verification

The engine (geodetic↔geocentric conversion, 7-parameter Helmert position-vector, bilinear and biquadratic grid interpolation with iterative inverse) was verified against PROJ 9.5 during the build: 161 test cases spanning grid shifts (forward and inverse), Helmert transformations, multi-grid NADCON5 chains, and full State Plane ftUS → State Plane ftUS pipelines. Worst disagreement: **0.002 mm**. Classic NADCON grids interpolate bilinearly and NADCON5 grids biquadratically, matching NGS/PROJ behavior.

Things to know:

- **Horizontal only.** Heights and any extra CSV columns pass through untouched. The grid format and pipeline were designed so vertical (NAVD88/GEOID) support can be added later as additional grid types.
- **NAD83 ↔ WGS 84 is inherently fuzzy.** "WGS 84" without an epoch is ambiguous at the sub-meter level. The EPSG operations offered treat it per their published definitions; the ITRF-based Helmert options are static snapshots (ITRF00, and ITRF2008 at epoch 2010.0) and ignore plate motion (~1–2 cm/yr). For sub-decimeter work across this pair, that's the limiting factor — same as in ArcGIS with the equivalent transformations.
- **Grid provenance:** NOAA/NGS grids from the OSGeo PROJ-data repository, converted losslessly (float32 preserved) into a compact deflate-compressed format (`.csg`) the browser can parse natively.

## Rebuilding or extending

The EPSG catalog and transformation list were generated from the EPSG database via pyproj and embedded as JSON inside `index.html` (search for `var CRS=` and `var OPS=`). To add a coordinate system, append an object with its proj4 string; to add a transformation, append an op with its steps (`h` = Helmert position-vector params, `g` = grid name, `c` = cartesian conversion with ellipsoid, `n` = null). Grid files can be regenerated from any PROJ GTG GeoTIFF with the ~40-line converter pattern: read bands (latitude_offset, longitude_offset in arcseconds), write the CSG header + float32 arrays, deflate.
