# AstraRoute: 3D Lunar Topography & Traversability Profiler

AstraRoute is an interactive, browser-based Web-GIS tool for exploring lunar terrain and planning surface routes. It converts a lunar digital elevation model (DEM) into a navigable 3D landscape, lets the user pick two points, and instantly produces a cross-sectional elevation profile, slope statistics, a traversability rating and an animated rover traverse with a simple travel-time and battery estimate.

The application opens with a real 12.2 km x 12.2 km tile of the Apollo 17 landing region (Taurus-Littrow valley) derived from NASA's Lunar Orbiter Laser Altimeter (LOLA) global elevation map.

**Live demo:** https://laraibkaleem06.github.io/astraroute/


> **Scope:** a prototype for visualization, education and demonstration. It is not validated for operational mission planning. See [Limitations](#limitations).

---

## Table of contents
1. [Overview](#overview)
2. [Features](#features)
3. [Quick start](#quick-start)
4. [User guide](#user-guide)
5. [Data](#data)
6. [Using your own DEM](#using-your-own-dem)
7. [Method](#method)
8. [Rover model](#rover-model)
9. [Mission report](#mission-report)
10. [Verification and validation](#verification-and-validation)
11. [Architecture](#architecture)
12. [Project structure](#project-structure)
13. [Browser support and performance](#browser-support-and-performance)
14. [Troubleshooting](#troubleshooting)
15. [Limitations](#limitations)
16. [Roadmap](#roadmap)
17. [Glossary](#glossary)
18. [Credits and licences](#credits-and-licences)
19. [Author](#author)

---

## Overview
Planning a rover traverse starts with a question: how steep and how long is the path between two places? Elevation data answers it, but raw rasters are hard to read. AstraRoute makes the data explorable:

- the terrain is rendered in 3D so craters, ridges and slopes are visible at a glance;
- two clicks define a route, and the tool samples the DEM along it to build a profile;
- derived metrics (distance, relief, slope) feed a green / amber / red traversability rating;
- a rover model drives the route, slowing on steep ground and tracking battery use, to show how terrain translates into time and energy.

Typical uses: teaching how DEMs are used in planetary science, demonstrating Web-GIS techniques, comparing candidate routes across a crater, and producing a one-page route summary.

## Features
**Terrain and navigation**
- 3D mesh built from a grayscale heightmap (256 x 256 grid), with rotate, pan and zoom on mouse and touch.
- Adjustable vertical exaggeration (1x to 6x), starfield backdrop and low-angle lighting.

**Route analysis**
- Click Point A and Point B on the surface; a glowing route is drawn on the terrain.
- Elevation profile chart: distance along the path (km) against elevation (m), with a hover marker that moves along the 3D route.
- Live metrics: total distance, peak elevation, lowest depth, maximum elevation difference, average slope (% and degrees) and steepest segment.
- Traversability status:

| Average slope | Status |
|---|---|
| below 15% | green: Safe Traversable Route |
| 15% to 25% | amber: Caution: Moderate Slope |
| above 25% | red: High Hazard / Steep Slope |

**Rover simulation**
- Animated rover that follows the terrain, turning and tilting with the ground.
- Slope-aware speed, live telemetry (distance, elapsed time, elevation, local slope, speed, battery), moving cursor on the chart and an optional follow camera.
- Travel and energy estimate: slope-aware versus flat-ground ETA, one-way energy, battery at the destination and a one-way / round-trip verdict.

**Data handling**
- Built-in real-data site (Apollo 17) and a terrain-source selector.
- Upload any grayscale DEM and set its elevation range and tile width.
- Save uploaded DEMs as named sites in the browser.

**Reporting**
- One-page report exported as PNG or PDF, including the 3D view, chart, metrics, rover estimate, data credit and disclaimer.

## Quick start
There is no build step and nothing to install. Open `index.html` in a modern browser with internet access; the charting, 3D and PDF libraries and the font load from public CDNs.

To publish with GitHub Pages: **Settings > Pages > Build and deployment > Deploy from a branch > `main` / `(root)` > Save**. The site is then served at `https://<username>.github.io/<repository>/`.

## User guide
**Camera.** Left-drag rotates, right-drag or Shift+drag pans, the mouse wheel zooms. On touch screens, one finger rotates and two fingers pinch to zoom.

**Drawing a route.**
1. Click the terrain to place Point A (cyan marker).
2. Click again to place Point B (purple marker). The route, chart and metrics appear immediately.
3. A third click starts a new route. **Reset Path** clears everything.
4. The coordinates of A and B are shown in the top-left box of the 3D view as kilometres east and south of the tile's north-west corner.

**Reading the results.**
- Hover over the chart to see the elevation at any distance; a white marker shows the matching spot on the 3D route.
- The status light and gauge reflect the average slope against the thresholds above.
- Peak, lowest and delta describe the relief along the route only, not the whole tile.

**Rover controls.**
- Play / Pause and Restart.
- Rover speed (1 to 20 km/h, default 10) sets the top speed on level ground.
- Time-lapse (1x to 300x, default 60x) speeds up the animation without changing the physics.
- Mass (kg) and Battery (kWh) feed the energy estimate.
- Follow camera keeps the view centred on the rover while still allowing rotation and zoom.

**Terrain source.** Choose between the built-in Apollo 17 tile, the synthetic demo terrain and any saved sites. The synthetic terrain is generated procedurally from fractal noise and random craters; it is clearly labelled and is not real lunar data.

## Data
**Built-in tile.** About 12.2 km x 12.2 km around the Apollo 17 landing site (approximately 20.2 N, 30.8 E), extracted from the NASA LRO LOLA global lunar DEM at 118 m per pixel (LOLA Science Team, NASA GSFC; USGS Astrogeology, 2014 mosaic).

**Processing applied.**
- Cropped to the area of interest.
- Resampled to 256 x 256 pixels. The native tile is only about 100 pixels wide, so it is enlarged.
- Converted to 8-bit grayscale over the tile's elevation range of -2925 m to -1135.5 m, which gives about 7 m of elevation per grey level.

**Reference surface.** Elevations are measured from the lunar reference sphere (radius 1737.4 km), so low terrain has negative values.

**Data quality notes.** Maps derived from LOLA laser tracks contain points that are interpolated where tracks leave gaps, so very small features may be estimated rather than measured.

### Reproducing the tile
Requires GDAL (bundled with QGIS; use the OSGeo4W Shell on Windows). The source map uses metre coordinates in a simple cylindrical projection, so the crop box is given in metres, not degrees.

```bash
# 1) Crop to the area of interest and convert to metres (Float32)
gdal_translate -unscale -ot Float32 -projwin 926590 618293 939508 606164 -outsize 256 256 \
  /vsicurl/https://planetarymaps.usgs.gov/mosaic/Lunar_LRO_LOLA_Global_LDEM_118m_Mar2014.tif apollo17.tif

# 2) Read the true minimum and maximum elevation
gdalinfo -stats apollo17.tif          # this tile: min -2925, max -1135.5

# 3) Convert to the 8-bit grayscale PNG used by the app
gdal_translate -of PNG -ot Byte -scale -2925 -1135.5 0 255 apollo17.tif apollo17.png
```

Resulting extent: 30 deg 33' E to 30 deg 59' E and 19 deg 59' N to 20 deg 23' N; the tile covers about 12.2 km in both directions.

**Choosing another site.** Multiply longitude (degrees east, 0 to 360) and latitude by about 30,323 to get metre coordinates for `-projwin`. One degree of longitude spans about 30.3 km x cos(latitude) on the ground, so widen the longitude range accordingly to keep the box square.

## Using your own DEM
1. Prepare an 8-bit grayscale PNG: black is the lowest elevation, white the highest. Square tiles work best; other shapes are stretched to 256 x 256.
2. In the sidebar, set **Min elev (m)**, **Max elev (m)** and **Tile width (km)** to the real values for the tile.
3. Click **Upload DEM (grayscale)** and select the file. If you change the numbers afterwards, click **Apply scaling**.
4. Optional: type a name and a data credit and click **Save site**. The site appears in the dropdown and reopens automatically next time. Saved sites live in the browser's local storage, so they stay on that browser and device.

The PNG stores no elevation values of its own. If the three numbers do not match the data, distances and slopes will be wrong.

## Method
**Height from brightness.** Each pixel's grey value (0 to 255) maps linearly to elevation:

```
elevation = min + (grey / 255) * (max - min)
```

**Spatial mapping.** The terrain is centred on the origin with 1 scene unit equal to 1 km. A click returns scene coordinates (x, z), which convert to fractional pixel coordinates (column, row). Ground size per pixel is `tile width / 255`.

**Height interpolation.** The elevation at a fractional pixel is the bilinear blend of its four surrounding cells, weighted by the fractional offsets, so the profile is continuous between pixels. The route from A to B is sampled at 200 equally spaced points along the straight line in pixel space.

**Slope.** For each sampled segment, `slope % = |rise| / run x 100`. The sidebar shows the mean and the maximum of these values; the angle uses `atan(mean / 100)`. This is the slope in the driving direction, which is never steeper than the terrain's steepest slope at the same point.

**Rendering.** Terrain heights are measured from the tile's mid-elevation so the mesh stays centred on the camera target for any site, positive or negative. Vertical exaggeration changes only the 3D display; the metrics, chart and report always use true elevations.

**Animation.** A `requestAnimationFrame` loop advances simulated time each frame (scaled by the time-lapse factor). The rover's position is found by interpolating between the 200 route samples using a precomputed arrival-time table, and its heading and pitch come from a short look-ahead along the route.

## Rover model
The rover model is intentionally simple and uses assumed values. These are not specifications of any real vehicle.

| Parameter | Value |
|---|---|
| Mass (default) | 1000 kg |
| Battery (default) | 10 kWh |
| Top speed (default) | 10 km/h (adjustable from 1 to 20) |
| Lunar gravity | 1.62 m/s^2 |
| Rolling resistance coefficient | 0.15 |
| Drive efficiency | 70% |
| Regeneration when descending | 30% |
| Base electrical load | 300 W |
| Battery reserve | 20% |

**Speed rules.** Uphill: full speed up to 8% grade, falling linearly to 25% of top speed at 25% and above. Downhill: full speed up to 15%, falling linearly to 40% of top speed at 30% and above, because braking rather than power limits descent.

**Energy per segment.**
```
force  = m * g * (mu * cos(theta) + sin(theta))
energy = force * segment length
```
Positive work costs `energy / efficiency`; negative work (descending) returns `energy x regeneration`. The base load multiplied by travel time is added to each segment.

**Verdict.** One-way energy and round-trip energy (the return leg uses the reversed slopes) are compared with 80% of the battery capacity, giving a green, amber or red status.

## Mission report
The **Export PNG** and **Export PDF** buttons produce a single dark-themed page (1600 x 1270 px) containing the 3D view with the route, the elevation chart, terrain metrics, the rover estimate with its assumptions, the data credit, the date and a disclaimer. Reports made on synthetic terrain are stamped as synthetic data.

## Verification and validation
**Built-in checks.**
- Horn's slope method applied to a plane with a known 10% gradient returns 5.71 degrees, matching `atan(0.1)`.
- Bilinear interpolation of a 2 x 2 grid at its centre returns the mean of the four corners.
- Energy model sanity values at 10 km/h and 1000 kg: a flat 5 km route costs about 0.63 kWh, a steady 20% climb over the same distance about 1.25 kWh (before speed reduction), and a steady 20% descent about 0.12 kWh because of regeneration.
- 8-bit quantization error stays within half a grey level (about 3.5 m for this tile).

**Comparison against NASA slope products.** `compare_slopes.py` checks the app's slope computation against an independent calculation and, optionally, against a NASA slope map for the same area (for example the LOLA-based slope products published on the LOLA PDS node). It reports:
1. the elevation error introduced by 8-bit storage,
2. the along-path slope as the app calculates it,
3. the steepest-direction slope from the DEM (Horn's method, as used by `gdaldem slope`),
4. when a NASA slope tile cut with the same box and size is supplied, the mean difference, RMSE and correlation along the route and across the whole tile.

```bash
python compare_slopes.py --dem apollo17.tif --slope apollo17_nasa_slope.tif \
  --width 12.2 --a 3.21 4.50 --b 8.70 7.10 --min -2925 --max -1135.5
```

`--a` and `--b` are the route points (kilometres east and south of the tile's north-west corner) shown in the app. The script needs Python with NumPy and GDAL.

Differences from NASA products are expected: they may combine different data sources and use their own baseline length, and a steepest-direction slope is always at least as large as an along-path slope.

## Architecture
The application is a single static page with no server or build tooling.

| Layer | Technology |
|---|---|
| Interface | HTML and custom CSS (glass panels, gradients, keyframe animations) |
| 3D rendering | Three.js r128: terrain mesh, route tube, rover, starfield, custom orbit/pan/zoom controls |
| Charting | Chart.js 4.4.1 with a custom plugin for the rover cursor |
| Report export | Canvas composition, jsPDF 2.5.1 for PDF |
| Data decoding | Canvas API reads pixel brightness as height |
| Persistence | Browser local storage (saved sites, last site) |
| Data preparation | GDAL (QGIS) and Python with NumPy |

Data flow: grayscale image, then height array, then terrain mesh and bilinear sampler, then route samples, then profile, metrics and rover plan, then chart, telemetry and report.

## Project structure
```
index.html          the complete application (HTML, CSS, JavaScript, built-in Apollo 17 tile)
README.md           this document
compare_slopes.py   slope validation script (optional)
```

## Browser support and performance
- Requires a modern desktop or mobile browser with WebGL and an internet connection for the CDN libraries and font.
- The page is about 50 KB; the built-in terrain is a 256 x 256 grid (about 65,000 vertices), which renders smoothly on typical laptops and recent phones.
- Saved sites rely on local storage and are not shared between browsers or devices.

## Troubleshooting
| Problem | Likely cause and fix |
|---|---|
| Blank 3D view | WebGL is disabled or blocked. Enable hardware acceleration, or try another browser. |
| Page loads but no styling or chart | CDN access blocked by an extension or network. Allow cdnjs.cloudflare.com and fonts.googleapis.com. |
| Distances or slopes look wrong for an uploaded DEM | Min, max or tile width do not match the data. Re-enter them and click **Apply scaling**. |
| Terrain looks stepped or terraced | 8-bit storage gives steps of about 7 m on this tile; this is a data-format limit, not a bug. |
| Export does not download | Check that the browser allows downloads from the site; try the other format. |
| Saved site missing | Local storage was cleared or a different browser or device is in use. |
| 404 on GitHub Pages | Wait a few minutes, confirm Pages is set to `main` / `(root)`, and confirm the file is named `index.html`. |

## Limitations
- **Resolution.** About 118 m per pixel: small craters, boulders and narrow ridges are not represented, and slopes are averaged over roughly 100 m.
- **Vertical precision.** 8-bit storage limits elevation to steps of about 7 m for the built-in tile.
- **Interpolated gaps.** Some points in the source map are interpolated rather than measured.
- **Rover model.** Simplified physics, assumed parameters and assumed slope-speed rules; the rover icon is not to scale.
- **Hazards not modelled.** Rocks, small craters, soil strength, sunlight and shadows, temperature and communications.
- **Straight-line routes.** Routes are straight lines between two points; there is no obstacle avoidance yet.
- **Thresholds.** The 15% and 25% slope limits are illustrative values, not mission criteria.

## Roadmap
- Slope heatmap overlay with legend
- Automatic safest-route finder (A* pathfinding) compared with the straight-line route
- Sun-angle lighting and shadow analysis
- Multi-waypoint routes and route comparison
- Higher-resolution tiles (for example 20 m per pixel near the lunar South Pole) with a site selector
- Published validation results against NASA slope maps

## Glossary
- **DEM (digital elevation model):** a raster in which each cell stores a height.
- **LOLA:** the Lunar Orbiter Laser Altimeter on NASA's Lunar Reconnaissance Orbiter, which measures surface height with a laser.
- **Heightmap:** a grayscale image used as a DEM, with brightness encoding elevation.
- **Bilinear interpolation:** estimating a value between four grid cells by weighted averaging.
- **Slope:** steepness, expressed as rise over run in percent (or as an angle).
- **Vertical exaggeration:** stretching heights in the 3D view to make relief easier to see.
- **Traversability:** how feasible it is for a vehicle to drive a route, here judged mainly by slope.
- **RMSE:** root-mean-square error, a measure of the typical difference between two datasets.

## Credits and licences
- **Elevation data:** NASA LRO LOLA global DEM, 118 m per pixel; LOLA Science Team, NASA GSFC; USGS Astrogeology (2014 mosaic). NASA and USGS data are generally free to reuse with credit; check the source pages for current terms.
- **Libraries:** Three.js (MIT), Chart.js (MIT), jsPDF (MIT).
- **Font:** Inter (SIL Open Font License).
- **Code licence:** MIT (see `LICENSE`).

