# Raw Data Sources

Raw input files are **not committed to this repository** (see root `.gitignore`) due to file size. Download from the sources below to reproduce this analysis.

## 1. Digital Elevation Model (DEM)

- **Source:** SRTM 30m or Copernicus GLO-30 DEM
- **Coverage needed:** Chocó, Risaralda, Caldas, Quindío departments, Colombia
- **Download:**
  - Copernicus GLO-30: https://portal.opentopography.org/raster?opentopoID=OTSDEM.032021.4326.3
  - SRTM: https://earthexplorer.usgs.gov/ (requires free USGS account)
- **Place in:** `data/raw/dem/`

## 2. USGS ShakeMap

- **Source:** USGS Earthquake Hazards Program
- **Event:** Colombia earthquake, August 10, 2026
- **Download:** https://earthquake.usgs.gov/earthquakes/ — search event by date/location, download ShakeMap grid (GeoTIFF or XYZ grid format)
- **Place in:** `data/raw/shakemap/`

## 3. OpenStreetMap road network

- **Source:** OpenStreetMap, via Overpass API or QuickOSM QGIS plugin
- **Query:** `highway` = `primary`, `secondary`, `tertiary` within study area bounding box
- **Recommended method:** QGIS QuickOSM plugin, or https://overpass-turbo.eu/ for a manual Overpass query, exported as GeoJSON
- **Place in:** `data/raw/osm/`

## 4. Validation reference (not GIS data — reporting only)

- INVÍAS: https://www.invias.gov.co/
- Servicio Geológico Colombiano (SGC): https://www.sgc.gov.co/
- UNGRD: https://www.gestiondelriesgo.gov.co/
- Copernicus Emergency Management Service: https://emergency.copernicus.eu/

---

*If any of these links have moved, search the organization name directly — Colombian government agency URLs change periodically.*
