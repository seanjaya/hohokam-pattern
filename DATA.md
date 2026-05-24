# Data sources & provenance

The three analysis-ready files in this repo — `tracts_working.gpkg`, `tracts_analyzed.gpkg`, and `tracts_analyzed_canopy.gpkg` — are derived from the five public sources below. The raw inputs aren't included (they're large and/or licensed), but every one is re-fetchable with the instructions here. Everything joins on **`GEOID`**, the 11-digit census-tract identifier (`04013XXXXXX`).

---

## 1. Canal network — Phoenix metro

- **Source:** USGS National Hydrography Dataset (NHD), High-Resolution, via The National Map ArcGIS REST API.
- **Filter:** FCode 33600 (Canal/Ditch) **plus** FCode 55800 (Artificial Path) where the GNIS name contains "Canal" or "Aqueduct" — the second clause is required or the Arizona Canal (SRP's flagship) is missed, because NHD codes through-flow canal segments as 55800.
- **Extent:** bounding box `-112.6, 33.2, -111.5, 33.9`.

```bash
curl -s "https://hydro.nationalmap.gov/arcgis/rest/services/nhd/MapServer/6/query?where=(FCODE%3D33600%20OR%20(FCODE%3D55800%20AND%20(GNIS_NAME%20LIKE%20%27%25Canal%25%27%20OR%20GNIS_NAME%20LIKE%20%27%25Aqueduct%25%27)))&geometry=-112.6,33.2,-111.5,33.9&geometryType=esriGeometryEnvelope&inSR=4326&spatialRel=esriSpatialRelIntersects&outFields=*&returnGeometry=true&outSR=4326&resultRecordCount=2000&f=geojson" -o nhd_canals_phoenix_metro.geojson
```

---

## 2. Census-tract boundaries — Maricopa County

- **Source:** US Census Bureau TIGER/Line Shapefiles 2024.
- **URL:** `https://www2.census.gov/geo/tiger/TIGER2024/TRACT/tl_2024_04_tract.zip`
- **Filter:** county FIPS `013` (1,009 tracts from the 1,765 statewide).

```python
import geopandas as gpd
url = "https://www2.census.gov/geo/tiger/TIGER2024/TRACT/tl_2024_04_tract.zip"
gdf = gpd.read_file(url)
maricopa = gdf[gdf["COUNTYFP"] == "013"]
maricopa.to_file("maricopa_tracts_2024.gpkg", driver="GPKG")
```

---

## 3. Urban heat island index by tract

- **Source:** Climate Central, *Urban Heat Hot Spots* (2023). A modeled index incorporating land cover, building height, and population density (methodology after Sangiorgio et al., 2020).
- **License:** **CC BY 4.0 — attribution is required in any published use.**
- **Coverage:** City of Phoenix only (~675 tracts), not the full metro. Of the 1,009 Maricopa tracts, ~673 join cleanly.
- **Note:** this is a modeled *index* (degrees of added heat from the built environment), not a measured surface temperature. It already absorbs green-space information.

Download the *Urban Heat Hot Spots* workbook from Climate Central and extract the Phoenix tracts (GEOID padded to 11 digits).

---

## 4. ACS demographics

- **Source:** US Census Bureau, American Community Survey 5-year 2023 estimates, via the Census API.
- **Requires a free API key:** https://api.census.gov/data/key_signup.html
- **Geography:** state `04`, county `013`, all tracts.

| Variable | Description |
| --- | --- |
| B01003_001E | Total population |
| B19013_001E | Median household income |
| B25077_001E | Median home value |
| B25064_001E | Median gross rent |
| B02001_002E | White alone |
| B02001_003E | Black alone |
| B03003_003E | Hispanic or Latino |
| B25035_001E | Median year structure built |
| B25003_002E | Owner-occupied housing units |
| B25003_001E | Total occupied housing units |

```python
import urllib.request, json, csv, os

API_KEY = os.environ["CENSUS_API_KEY"]   # keep your key out of the repo
variables = ["NAME","B01003_001E","B19013_001E","B25077_001E","B25064_001E",
             "B02001_002E","B02001_003E","B03003_003E","B25035_001E",
             "B25003_002E","B25003_001E"]
url = (f"https://api.census.gov/data/2023/acs/acs5?get={','.join(variables)}"
       f"&for=tract:*&in=state:04%20county:013&key={API_KEY}")
with urllib.request.urlopen(url) as r:
    raw = json.load(r)
with open("acs_2023_5yr_maricopa_tracts.csv", "w", newline="") as f:
    w = csv.writer(f); w.writerow(raw[0] + ["GEOID"])
    for row in raw[1:]:
        state, county, tract = row[-3], row[-2], row[-1]
        w.writerow(row + [state + county + tract])
```

---

## 5. Tree canopy by tract

- **Source:** MRLC National Land Cover Database (NLCD) Tree Canopy Cover, 30 m raster.
- **Method:** fetched for the area of interest via `pygeohydro`, then aggregated to tracts with a `rasterstats` zonal mean.
- **Caveat:** NLCD tree-canopy reads low in arid cities (tract means < 1%); use it for relative comparison, not as an absolute canopy estimate.

---

## Processed files in this repo

| File | Produced by | Contents |
| --- | --- | --- |
| `tracts_working.gpkg` | `01_eda.ipynb` | Cleaned, joined tract geometries + base features |
| `tracts_analyzed.gpkg` | `02_analysis.ipynb` | Features, cluster labels, archetypes — the frozen single source of truth |
| `tracts_analyzed_canopy.gpkg` | `03_canopy_mediation.ipynb` | Above + tree-canopy fraction for the mediation test |
