# The Hohokam Pattern

### Canal Proximity, Urban Heat, and the Geography of Who Stays Cool in Phoenix

**Technical Report**
Author: Sean Jayasekera
Date: May 2026
Project: *somewhere-else.org* — Week 3

---

## Abstract

Phoenix sits on a canal network whose lineage runs from the Hohokam civilization through the modern Salt River Project. A natural hypothesis is that proximity to flowing water cools the surrounding city. This report tests that hypothesis at the census-tract level across the central Phoenix metropolitan area and finds that it does not hold: distance to the nearest canal and canal density within a tract are essentially uncorrelated with the urban heat island (UHI) index. Instead, the strongest single predictor of tract-level heat is the median year structures were built — a proxy for the age and form of the built environment — followed by measures of wealth and tenure. A four-cluster typology of neighborhoods recovers a more textured result: among the city's Hispanic-majority tracts, those with high canal density run measurably cooler than those without, consistent with the persistence of flood irrigation in a small set of older working-class neighborhoods. The canal network does not cool Phoenix as infrastructure; it cools the specific places that still use it the old way. The heat burden otherwise falls along the lines of when a neighborhood was built and who was historically able to settle there.

---

## 1. Background

The Phoenix metropolitan area is among the most intensely heat-affected large urban regions in the United States, and the urban heat island effect — the tendency of dense, impervious, low-canopy development to retain and re-radiate daytime heat — is a central feature of life there. The region is also defined by water infrastructure to an unusual degree. The Hohokam built hundreds of miles of irrigation canals across the Salt River valley beginning more than a thousand years ago; the modern canal system, operated principally by the Salt River Project (SRP) and the Central Arizona Project (CAP), in many places follows or reuses those ancient alignments.

The intuition that canals cool their surroundings is reasonable on its face. Open water moderates temperature through evaporation, and the vegetation and irrigation historically associated with canal corridors add shade and evapotranspiration. If that mechanism operated at the scale of the city, we would expect tracts nearer to or denser in canals to register lower UHI values. This report tests that expectation directly and then, having found it unsupported at the aggregate level, asks what does explain the spatial pattern of urban heat in Phoenix.

---

## 2. Research Question

The analysis is organized around one primary question and two secondary questions that emerged as the primary result became clear.

**Primary.** Does proximity to, or density of, the canal network predict lower tract-level urban heat island intensity in the Phoenix metro?

**Secondary, on substitution.** If canals do not explain the heat pattern, which tract characteristics do?

**Secondary, on heterogeneity.** Is the canal–heat relationship uniform across neighborhood types, or does it appear only in particular kinds of places?

---

## 3. Data

The analysis integrates four sources, joined at the census-tract level.

**Canal network — USGS National Hydrography Dataset (NHD).** Canal and aqueduct features were queried from the NHD MapServer for the Phoenix metro extent, yielding 1,475 features spanning five operating authorities (SRP, CAP, Roosevelt Water Conservation District, Maricopa Water District, and Buckeye Valley Irrigation District). A notable data-quality finding shaped the extraction: the Arizona Canal — one of the most significant canals in the system — is classified in NHD as FCode 55800 ("Artificial Path") rather than the canal-specific FCode 33600. A naive filter on the canal FCode alone would have silently dropped it. The final query combined the FCode filter with a name-based filter to recover all major named canals and aqueducts.

**Census geography — TIGER/Line 2024 tracts.** All 1,009 census tracts in Maricopa County were obtained from the Census Bureau's 2024 TIGER/Line release, in geographic coordinates (EPSG:4269, NAD83). Tract polygons supply both the unit of analysis and the basis for area and proximity computations.

**Urban heat — Climate Central Urban Heat Hot Spots (2023).** Tract-level UHI index values were drawn from Climate Central's 2023 urban heat assessment, which provides a modeled UHI index in degrees Fahrenheit for 675 tracts. An important scoping note: although the dataset is labeled for the "City of Phoenix," its spatial extent (roughly 98 by 58 miles) covers the central Phoenix metropolitan area as a whole, including Scottsdale, Glendale, Tempe, and Mesa. The report uses "Phoenix metro" or "the study area" rather than "the City of Phoenix" to describe the analytical universe, and the figure of 675 tracts with UHI coverage defines that universe.

**Demographics and housing — American Community Survey, 2023 5-year estimates.** Tract-level estimates for all 1,009 Maricopa tracts were pulled from the Census API, covering total population, median household income, median home value, median gross rent, median year structure built, and the percentages of residents who are White, Black, and Hispanic, and of housing units that are owner-occupied. Two ACS top-coding conventions are relevant downstream: median household income is top-coded at \$250,001 and median home value at \$2,000,001. These ceilings compress the upper tail of both variables and attenuate any correlation involving them, a point returned to in the limitations.

After joining the four sources and restricting to tracts with valid UHI coverage, the working universe comprises 671 tracts (after the removal of two extreme-area outliers described in Methods). Subsequent analytical steps operate on slightly different subsets depending on missing-data requirements, stated explicitly throughout.

---

## 4. Methods

### 4.1 Spatial feature construction

All distance and area computations were performed in a projected coordinate system (EPSG:2223, NAD83 Arizona Central State Plane, US feet) appropriate for accurate measurement in the study area; demographic joins and final storage use geographic coordinates. For each tract, the analysis computed: distance to the nearest canal feature (`nearest_canal_mi`), total canal length falling within the tract (`canal_mi_in_tract`), canal density (`canal_per_mi2`, total canal length normalized by tract area), and tract area (`area_mi2`). Two tracts with anomalous areas exceeding roughly 85 and 115 square miles — corresponding to the CAP aqueduct's passage through largely federal and tribal land — were removed before correlation analysis to prevent geometric outliers from distorting the canal-length measure.

### 4.2 Dual-track inference

Two-group comparisons were conducted under both frequentist and Bayesian frameworks, in parallel, so that conclusions do not rest on the conventions of a single school of inference. The frequentist track uses Welch's t-test (unequal variances), reporting the mean difference, a 95% confidence interval, the t-statistic, the two-sided p-value, and Cohen's d as a standardized effect size. The Bayesian track fits a Welch-analog model in PyMC — independent normal likelihoods for each group with weakly informative priors on the means (Normal, centered at the pooled mean with wide variance) and half-normal priors on the group standard deviations — and samples the posterior via MCMC (four chains, 1,000 tuning and 2,000 draw iterations each). It reports the posterior mean difference, a 95% credible interval, the posterior distribution of Cohen's d, and directly interpretable posterior probabilities such as P(difference < 0). Convergence was assessed by the Gelman–Rubin statistic and effective sample size.

### 4.3 Clustering and dimensionality reduction

To characterize neighborhood types, the analysis applied K-means clustering to six standardized tract features chosen to describe what *kind* of place a tract is rather than how hot it is: median year built, median household income, percent owner-occupied, percent Hispanic, canal density, and tract area. The UHI index was deliberately excluded from the clustering features so that heat could be examined as an *outcome* across the resulting clusters rather than baked into their definition. Features were standardized to zero mean and unit variance so that no single variable's scale dominated the Euclidean distance metric. The number of clusters was selected with reference to both the inertia elbow and the silhouette score; K = 4 was chosen to balance statistical fit against the interpretive value of recovering distinct archetypes. Principal component analysis on the same standardized features provided a two-dimensional projection for visualization and a reading of which features define the major axes of variation.

### 4.4 Analytical subsets

Because different methods have different missing-data tolerances, three subsets are used and named throughout:

- **Correlation and canal A/B tests:** 671 tracts (the working universe after extreme-area removal).
- **Year-built A/B test:** 646 tracts (after dropping tracts missing median year built).
- **Clustering and PCA:** 642 tracts (after restricting to tracts with area ≤ 50 mi² and complete data on all six clustering features). This step removed four further large-area tracts that K-means otherwise isolated as a degenerate single-tract cluster.

---

## 5. Results

### 5.1 The canal hypothesis is not supported

At the aggregate level, canal proximity does not predict urban heat. Distance to the nearest canal is effectively uncorrelated with the UHI index (Pearson r = −0.070, p = 0.069). The Bayesian A/B test comparing high-canal-density tracts against low-canal-density tracts returns a posterior mean difference of just −0.045 °F with a 95% credible interval of [−0.19, +0.094] °F — an interval that comfortably spans zero — and a posterior probability that dense tracts are cooler of only 0.74. The standardized effect size is negligible (Cohen's d = −0.049). Even the most favorable possible comparison, between the top and bottom quartiles of canal density, yields a difference of only −0.117 °F (p = 0.168, Cohen's d = −0.134) with a confidence interval that still crosses zero.

One canal metric does show a non-trivial correlation: total canal length within a tract (`canal_mi_in_tract`, r = −0.266). This figure should not be read as support for the cooling hypothesis. Raw canal length within a tract is confounded with tract geography — larger and more centrally located tracts tend to contain more canal mileage for reasons unrelated to any cooling mechanism — and the two cleaner measures that isolate the proximity question, distance-to-nearest and area-normalized density, are both null. The A/B test on canal density, the appropriate test of the hypothesis, confirms a trivial effect. The canal hypothesis, as a claim about the city as a whole, does not survive.

### 5.2 Year built is the dominant predictor

Ranking all candidate tract characteristics by the absolute strength of their correlation with the UHI index produces a clear ordering:

| Rank | Variable | Pearson r with UHI |
|---|---|---|
| 1 | Median year built | **−0.485** |
| 2 | Percent owner-occupied | −0.384 |
| 3 | Median household income | −0.340 |
| 4 | Median gross rent | −0.327 |
| 5 | Canal length in tract | −0.266 |
| 6 | Median home value | −0.149 |
| 7 | Percent Hispanic | +0.137 |
| 8 | Percent White | −0.122 |
| — | Tract area, distance to canal, percent Black, canal density | trivial |

Median year built is the strongest single predictor, accounting for roughly 24% of the variance in tract-level UHI on its own. The sign is negative: newer tracts run cooler. The A/B test formalizes this. Splitting tracts at the median construction year (1985) into older (built ≤ 1985, n = 333, mean UHI 7.81 °F) and newer (built > 1985, n = 313, mean UHI 7.12 °F) groups:

| Framework | Mean difference (newer − older) | 95% interval | Effect size (Cohen's d) |
|---|---|---|---|
| Frequentist (Welch) | −0.698 °F | [−0.826, −0.570] | −0.853 |
| Bayesian (PyMC) | −0.697 °F | [−0.827, −0.569] | −0.845 |

The two frameworks agree to the second decimal place. The effect is large by Cohen's conventions (|d| > 0.8), the interval is tight and entirely below zero, and the frequentist p-value (2.2 × 10⁻²⁴) and Bayesian posterior probability that newer tracts are cooler (1.000) are about as decisive as either framework produces. The posterior further indicates a 99.8% probability that the effect is at least half a degree, and essentially zero probability that it reaches a full degree — bounding the claim cleanly: newer tracts are cooler than older ones by somewhere between roughly half a degree and the better part of one degree, on average.

The distributional shapes carry an additional nuance. Older tracts form a tight, tall distribution peaked near 8 °F — they are reliably hot, with little spread. Newer tracts form a broader, lower distribution: some new development is as hot as the old core, but its floor is far lower. Newer construction does not guarantee a cool tract; it makes a cool tract possible.

### 5.3 A four-archetype neighborhood typology

K-means clustering on the six neighborhood-characteristic features (K = 4, n = 642) recovers four interpretable archetypes. Reading them from hottest to coolest:

| Archetype | n | Median yr built | Median income | % Hispanic | Canal density (per mi²) | Mean UHI (°F) |
|---|---|---|---|---|---|---|
| Older Hispanic urban core | 232 | 1975 | \$61K | 50% | 0.35 | **7.91** |
| Working-class Hispanic, canal-adjacent | 143 | 1990 | \$76K | 48% | **2.18** | 7.33 |
| Newer middle/upper-class suburbs | 260 | 1990 | \$110K | 16% | 0.39 | 7.23 |
| Outer-ring exurbs | 7 | 2003 | \$123K | 8% | 0.10 | **6.03** |

The typology reproduces the variable-level findings in spatial form. The hottest archetype is the older, lower-income, Hispanic-majority urban core; the coolest is the newest, wealthiest, least diverse exurban fringe (though this last cluster is small, n = 7, and is best read as a directional signal rather than a robust group). Critically, canal density does not track the heat ranking: the cluster with by far the highest canal density (the canal-adjacent cluster, at 2.18 canal-miles per square mile, roughly six times the others) sits in the *middle* of the UHI range, not at the cool end, while the coolest substantial cluster has among the lowest canal density.

Principal component analysis clarifies why. The first two components capture 57.5% of the variance in the six features. PC1 (39.0%) is a composite wealth-and-era axis — income, owner-occupancy, and year built load positively, percent Hispanic loads negatively. PC2 (18.5%) is dominated by canal density, which carries a loading of +0.819, the single largest loading on any component, and points nearly orthogonally to the wealth axis. In plain terms: how many canals a tract contains varies almost independently of how wealthy, new, or demographically composed it is. Canals are their own dimension of the city, and that dimension is not the one that governs heat.

### 5.4 The flood-irrigation exception

The clustering also recovers the result that turns a null finding into a story. The two Hispanic-majority archetypes — the older urban core and the canal-adjacent cluster — are similar in ethnic composition (50% versus 48% Hispanic) and broadly comparable in income, but differ sharply in canal density (0.35 versus 2.18). The high-canal cluster runs 0.58 °F cooler in the mean (7.33 versus 7.91 °F). Moreover, its UHI distribution is left-skewed — its mean (7.33) sits below its median (7.55), indicating a tail of notably cool tracts pulling the average down.

The most plausible explanation is flood irrigation. A subset of older working-class neighborhoods in Phoenix retain the practice of flood-irrigating yards and lots directly from the canal system, a holdover from the area's agricultural past. Where that practice persists, the added soil moisture, vegetation, and evapotranspiration cool the immediate environment. The canal network, in other words, does cool the city — but only in the specific places that are still plumbed into it for irrigation rather than merely adjacent to it as infrastructure. This is consistent with the aggregate null: most tracts near canals are not flood-irrigated, so proximity alone carries no signal, and the cooling concentrates in a minority of tracts that the city-wide test cannot detect but the clustering can isolate.

---

## 6. Limitations

**Ecological inference.** All findings are at the level of the census tract, not the individual or the household. A correlation between a tract's demographic composition and its average heat exposure does not describe any individual resident's experience, and inferences from aggregate units to individuals would commit the ecological fallacy. The demographic findings should be read as statements about *places* and the historical processes that shaped them, not about people.

**Correlation, not causation.** The design is observational. Year built, wealth, and demographic composition are deeply intertwined with one another and with unmeasured factors — tree canopy, impervious surface, building density, lot size — that are the more proximate physical drivers of urban heat. Year built is best understood as a proxy for the form of the built environment characteristic of the era in which a neighborhood was developed, not as a cause in itself. The analysis identifies which tract characteristics predict heat; it does not establish the causal pathway.

**Demographic composition as marker, not mechanism.** The finding that Hispanic-majority tracts run hotter does not imply that ethnicity causes heat. It reflects the historical geography of Phoenix's development: which neighborhoods were built when, with what infrastructure and canopy investment, and which communities were able to settle where over decades of housing and development policy. The heat burden tracks the legacy of that geography. Read this way, the result is an environmental-justice observation about the unequal distribution of a climate harm, and the flood-irrigation exception points toward a concrete and retainable form of mitigation already present in some of the affected neighborhoods.

**Top-coding.** ACS top-codes median household income at \$250,001 and median home value at \$2,000,001. Both ceilings compress the upper tail and attenuate the measured correlations involving wealth; the true association between wealth and coolness is likely somewhat stronger than the reported coefficients suggest. This makes the home-value correlation (r = −0.149) in particular a conservative lower bound.

**Weak clustering structure.** Silhouette scores across all candidate values of K remained in the 0.23–0.26 range, indicating that Phoenix neighborhoods vary along a continuum rather than falling into naturally separable groups. The four archetypes are therefore best described as a useful summary typology constructed by the analyst, not a discovered taxonomy that exists independently in the data. They are interpretive scaffolding, and the report frames them as such.

**Small exurban cluster.** The coolest archetype contains only seven tracts. Its direction is consistent with the overall pattern, but its specific values should be treated as indicative rather than precise.

**UHI definition.** The UHI index is Climate Central's modeled 2023 product with its own methodology and spatial extent. Findings are specific to that operationalization of urban heat and to the central-metro tracts it covers.

---

## 7. Conclusion

Phoenix's canals do not cool the city — except where they irrigate it. Tested as a city-wide proposition, canal proximity and canal density carry essentially no signal for urban heat; the relationship that the intuition predicts simply is not there in the aggregate. What predicts heat instead is the age and form of the built environment, captured most strongly by median year built, together with the wealth and tenure measures that move with it. The heat burden in Phoenix falls along the lines of when neighborhoods were built and, inseparably, of who was historically able to settle in them — concentrated most heavily and most reliably in the older, lower-income, Hispanic-majority urban core.

Yet the canal lineage that gives the city its shape has not vanished from the data. Inside those same Hispanic-majority neighborhoods, the tracts still connected to the canal system through flood irrigation run measurably cooler than the tracts that are not. The Hohokam pattern survives in two places at once: in the canal network itself, which the whole city still depends on for water, and in the small set of working-class neighborhoods that still use the canals the old way — and are cooler for it. The thousand-year-old infrastructure does its cooling work not where the water passes by, but where the water is still let out onto the land.

---

## Appendix: Reproducibility

All analysis is contained in two Jupyter notebooks. Notebook 01 performs exploratory data analysis and cleaning, producing `data/processed/tracts_working.gpkg`. Notebook 02 performs the core analysis end-to-end and produces `data/processed/tracts_analyzed.gpkg` (671 tracts with all features, cluster assignments, and archetype labels) and an accompanying CSV. A fixed random seed (42) governs all stochastic steps — MCMC sampling, K-means initialization, and PCA — so results are reproducible across runs. Every figure and statistic in this report traces to a specific, numbered cell in Notebook 02, which is the single source of truth for the project's numbers.
