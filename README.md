# The Hohokam Pattern

> Phoenix is built on a thousand-year-old canal network — so the neighborhoods nearest the water should be the coolest. They aren't. This is an analysis of what actually decides who stays cool in Phoenix, and the one place the old canals still do their job.

![The Hohokam Pattern](article/hohokam_pattern_cover.png)

A spatial analysis of urban heat across the Phoenix metro at the census-tract level. The guiding question was whether proximity to the canal network keeps neighborhoods cooler. It does not — at the city scale, canal proximity has no measurable effect on heat. What predicts heat is **when a neighborhood was built and who has historically lived there**, with the burden concentrated in the oldest, lowest-income, Hispanic-majority core. The exception is a set of working-class neighborhoods that still **flood-irrigate** from the canals; there the water measurably cools the block — and the mechanism is direct evaporative cooling at ground level, **not** tree shade.

**Read it:** [the article](https://somewhere-else.org/) · [the technical report](report/technical_report.pdf)

## Key findings

- **The canal null.** Distance to the nearest canal vs. heat: r = −0.07 (not significant). The heat gap between the densest and sparsest canal coverage is ≈ 0.05°F — statistical noise.
- **Build era is the strongest single predictor.** Neighborhoods built before ~1985 run ≈ 0.70°F hotter than those built after (Cohen's *d* ≈ −0.85, large) — roughly 17× the canal effect.
- **Four neighborhood archetypes** (K-Means). Hottest: the older Hispanic urban core (7.91°F). Coolest: new outer-ring exurbs (6.03°F).
- **The flood-irrigation exception.** Two demographically near-identical Hispanic clusters differ by 0.58°F; the canal-fed, flood-irrigated one is cooler.
- **The mechanism is water, not shade.** Tree canopy does not mediate the cooling — holding canopy constant moves the gap by 0.003°F (canopy coefficient n.s.). The cooling is ground-level evaporative.

## Data

| Source | Provides | Notes |
| --- | --- | --- |
| USGS NHD (The National Map) | Canal / ditch network | High-resolution hydrography, FCode-filtered |
| Census TIGER/Line 2024 | Census-tract geometries | Maricopa County (FIPS 04013) |
| Climate Central UHI (2023) | Urban heat island index by tract | **CC BY 4.0 — attribution required**; City of Phoenix only (~675 tracts) |
| Census ACS 5-year 2023 | Demographics — year built, income, tenure, ethnicity | Requires a free Census API key |
| MRLC NLCD Tree Canopy | Tree-canopy % by tract | 30 m raster, zonal mean |

Full provenance, endpoints, and re-fetch instructions are in [`data/README.md`](data/README.md).

## Reproducing the analysis

```bash
conda env create -f environment.yml
conda activate hohokam-pattern
jupyter lab
```

Run the notebooks in order:

1. **`notebooks/01_eda.ipynb`** — cleaning, spatial joins, exploratory analysis → `data/processed/tracts_working.gpkg`
2. **`notebooks/02_analysis.ipynb`** — feature engineering, K-Means clustering, frequentist + Bayesian hypothesis tests → `data/processed/tracts_analyzed.gpkg` *(the frozen single source of truth)*
3. **`notebooks/03_canopy_mediation.ipynb`** — tree-canopy mechanism test → `data/processed/tracts_analyzed_canopy.gpkg`

Conventions worth knowing: the random seed is fixed at **42**; the analysis CRS is **EPSG:2223** (Arizona Central, US ft) and the storage CRS is **EPSG:4269**. Notebook 02's outputs are treated as frozen and feed everything downstream. The ACS step needs a Census API key (free, ~1 minute); the canopy step downloads an NLCD raster and needs network access.

## Repository structure

```
hohokam-pattern/
├── README.md
├── LICENSE
├── environment.yml
├── .gitignore
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_analysis.ipynb          # + 02_analysis.html (rendered)
│   └── 03_canopy_mediation.ipynb  # + 03_canopy_mediation.html (rendered)
├── data/
│   ├── README.md                  # source provenance + re-fetch instructions
│   ├── raw/                       # gitignored — fetched, not committed
│   └── processed/                 # committed (small, tract-level)
│       ├── tracts_working.gpkg
│       ├── tracts_analyzed.gpkg / .csv
│       └── tracts_analyzed_canopy.gpkg / .csv
├── report/
│   ├── technical_report.md
│   └── technical_report.pdf
└── article/
    ├── hohokam_pattern_article.html
    └── hohokam_pattern_cover.png
```

## Caveats

- The analysis is **correlational and ecological** (tract-level). Associations are not individual-level causal claims.
- The heat variable is a **modeled UHI index**, not measured surface temperature, and covers the City of Phoenix, not the full metro (~673 tracts join cleanly).
- NLCD tree-canopy reads low in arid cities (tract means < 1%); canopy is used for **relative comparison and as a mediator test**, not as an absolute estimate.

## License

Code is released under the MIT License (see `LICENSE`). Data is subject to each provider's terms — in particular, the Climate Central UHI data is **CC BY 4.0 and requires attribution** in any published use.
