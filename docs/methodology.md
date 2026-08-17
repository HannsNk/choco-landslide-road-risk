# Methodology

## 1. Study area

Chocó and the surrounding coffee-growing departments (Risaralda, Caldas, Quindío), Colombia — centered on the epicenter region of the August 10, 2026 earthquake. Mountainous terrain with dense secondary/tertiary road networks connecting rural communities.

## 2. Inputs

| Layer | Source | Resolution / format | CRS |
|---|---|---|---|
| DEM | SRTM / Copernicus GLO-30 | ~30m | Reprojected to EPSG:32618 (UTM 18N) |
| Slope | Derived from DEM (QGIS `Slope`, GDAL) | ~30m | EPSG:32618 |
| Ground shaking intensity | USGS ShakeMap, Aug 10 2026 event | Native ShakeMap grid, rasterized | EPSG:32618 |
| Road network | OpenStreetMap, `highway` = primary\|secondary\|tertiary | Vector (lines) | Reprojected 4326 → 32618 |

All raster inputs were reprojected/resampled to a common CRS (EPSG:32618, UTM Zone 18N — appropriate for this longitude band and needed for metric distance/area calculations) and aligned to matching cell size before combination.

## 3. Risk index construction

Both slope and shaking intensity rasters were **normalized to a 0–1 range** prior to combination, to prevent one input's native units/scale from dominating the index purely due to magnitude rather than actual relative importance.

```
risk_index = normalized_slope × normalized_shaking_intensity
```

A multiplicative combination was used deliberately: this reflects that landslide susceptibility from ground shaking requires *both* steep terrain *and* strong shaking — flat ground under strong shaking is low risk, and steep terrain under weak shaking is also low risk. An additive index would incorrectly assign moderate risk to areas strong in only one factor.

Resulting continuous index range: **0 to ~73** (unitless, relative index — not a probability or engineering safety factor).

## 4. Classification into tiers

The combined index distribution is **heavily right-skewed**: the large majority of pixels fall below ~5, with a long tail extending to ~73. This shape is expected — most terrain in the study area is not simultaneously both steep and strongly shaken.

Given this skew, **equal-interval-style approximated buckets** (rather than computed percentiles) were chosen, iterated against histogram inspection and visual map checks in QGIS:

| Tier | Range | Interpretation |
|---|---|---|
| 0 | 0–5 | Negligible |
| 1 | 5–15 | Low |
| 2 | 15–30 | Moderate |
| 3 | 30–45 | **Medium-high** (included in road analysis) |
| 4 | 45–100 | **High** (included in road analysis) |

Tiers 0–1 were dropped before vectorization — both for interpretive relevance (not meaningfully "at risk") and for practical reasons (tier 1 alone contained ~916,000 raw pixel features prior to filtering, which is not a tractable vector layer).

> **Note on methodological rigor:** an earlier draft of this analysis considered citing a specific MMI-6 / 15°-slope threshold as an established landslide-triggering criterion. On review, this was found to be an unsourced claim rather than an established standard, and was removed. Rigorous landslide-triggering thresholds (e.g. **Newmark sliding-block analysis**, **Arias intensity**) require geotechnical inputs — soil cohesion, friction angle, saturation state — that are not available from the open datasets used here. The tiering above should be read as a **relative susceptibility proxy**, not a calibrated triggering threshold.

## 5. Vectorization

- **GDAL Polygonize**, 4-connectedness (not 8-connectedness — avoids merging diagonally-adjacent same-tier cells that may represent genuinely distinct terrain features).
- A continuous raster **must** be reclassified into discrete integer bins *before* polygonizing — polygonizing a continuous raster directly produces a single enormous multi-part polygon with holes, since GDAL Polygonize groups by exact matching pixel value.
- Features under **5,000 m²** (~5.6 pixels at 30m resolution) were filtered out as likely sub-pixel classification noise rather than genuine contiguous risk zones.

## 6. Road intersection

- OSM road layer reprojected from EPSG:4326 → EPSG:32618 to match the risk layer.
- Vector **Intersection** of tier 3/4 polygons against the road layer.
- Result: 366 features, each retaining both `highway` (road class) and `risk_tier` fields — allowing prioritization by road importance × risk severity (e.g. a primary road in tier 4 is a higher priority than a tertiary road in tier 3).
- Tier 3/4 polygons were confirmed, by visual inspection, to be genuinely spatially disconnected rather than a 4-connectedness artifact — so a dissolve step was not analytically necessary (a dissolve was attempted but abandoned due to memory constraints on the working machine; determined afterward to be unneeded).

## 7. Validation approach

No calibrated ground-truth landslide inventory was available for this project. As a **qualitative sanity check only**, resulting high-risk road segments were cross-referenced against contemporaneous reporting on post-earthquake road closures in the region. Authoritative sources identified for anyone wishing to validate more rigorously:

- **INVÍAS** (Instituto Nacional de Vías) — national road authority, closure/status reporting
- **SGC** (Servicio Geológico Colombiano) — geological hazard mapping
- **UNGRD** (Unidad Nacional para la Gestión del Riesgo de Desastres) — disaster response coordination
- **Copernicus EMS** (Emergency Management Service) — satellite-derived rapid damage assessment, when activated for an event

This cross-check is **not a substitute for quantitative validation** and should not be read as confirming the model's accuracy — only that its outputs are broadly plausible given real-world reporting.

## 8. QGIS-specific workflow notes (for ArcGIS Pro users)

| ArcGIS Pro concept | QGIS equivalent used |
|---|---|
| Spatial Analyst → Slope | `Slope` (GDAL/native) |
| Reclassify | `Reclassify by table` (raster) |
| Raster to Polygon | `Polygonize (Raster to Vector)` (GDAL) |
| Intersect | `Intersection` (vector overlay) |

## 9. Known limitations (see also README)

- Proxy index, not a geotechnically validated model.
- No landslide inventory for quantitative calibration/accuracy assessment.
- 30m DEM resolution limits detection of small-scale slope failures.
- ShakeMap represents modeled/interpolated shaking intensity, not directly observed ground motion at every point.
