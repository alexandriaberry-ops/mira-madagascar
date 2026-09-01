# MIRA Madagascar — Vegetation Indices and Household Food Consumption

Data and analysis code accompanying the MIRA study of southern Madagascar,
relating satellite vegetation indices (EVI and NDVI) to household food
consumption at the commune level.

The analysis reduces per-item household consumption records to a single
dominant dimension of dietary variation via mean-centered singular value
decomposition, then tests that component against commune-level vegetation
greenness over time.

> *Recurrent droughts and food shortages in southern Madagascar underscore the need for real-time food security monitoring.
Since 2018, a monthly survey of 602 households has tracked dietary coping strategies in response to food shocks; 547
meeting a 70% response-rate threshold are included in our analysis. Here we assess whether satellite-measured vegetation
greenness can track variation in household-level food consumption. Greenness does not predict the four commonly used food
security indices in the survey. It is instead significantly associated with the leading mode of dietary variability, which captures
substitution between staple or aid-supported foods and lower-preference fallback foods eaten during food stress. Projected
onto commune-level monthly patterns, this mode explains an average of 43% of within-commune dietary variability. Greenness
represents it skillfully, explaining 32–58% of its variability, and 38–70% once seasonality is included. Higher-order dietary
modes show no such relationship, and rotated decompositions, lagged predictors, and alternative vegetation indices do not
improve performance. Satellite-measured greenness therefore captures primary dietary shifts, supporting its integration into
more-detailed food security monitoring systems.

DOI added once available.

## Repository layout

```
data/
  mira_cleaned_git.csv          per-household food consumption, de-identified
  commune_food_monthly.csv      mean food consumption by commune-month
  commune_monthly.csv           monthly NDVI and PC1 score by commune
  commune_lookup.csv            commune code, name, original shapefile code
  evi_timeseries/               commune-mean EVI series, communes 1-6
  ndvi_timeseries/              commune-mean NDVI series, communes 1-6
  boundaries/                   Madagascar admin boundaries, ADM0-ADM4
  README.md                     full data dictionary
code/
  create_commune_svd_vecs.ipynb SVD of consumption data; writes commune PC1 vectors
  EVI_SVD_Mean_ctrd.ipynb       SVD of consumption data; DC1 vs. EVI
  svd_ndvi_commune_mean_centering.ipynb  DC1 vs. NDVI; correlations and panel figure
  dc1_commune_models.ipynb      per-commune OLS models, NDVI vs. PC1
requirements.txt
CITATION.cff
```

## Study communes

Every file in this repository uses one commune numbering, **1-6**, matching the
manuscript figures (ordered left to right across the study area). Communes were
originally coded by their position in the source shapefile; that correspondence
is recorded in `data/commune_lookup.csv` and reproduced here for reference.

| Code | Commune | Original shapefile code |
|---|---|---|
| 1 | Marolinta | 2 |
| 2 | Tranovaho | 1 |
| 3 | Marovato | 6 |
| 4 | Anjampaly | 5 |
| 5 | Antaritarika | 7 |
| 6 | Imongy | 8 |

Shapefile communes 3 (Nikoly) and 4 (Betanty / Faux Cap) were collapsed into
Anjampaly (code 4) during survey aggregation. Their households are included in
that commune's food data; the vegetation series for code 4 covers the Anjampaly
polygon only.

No released file uses the original codes, so `commune` joins directly across
`commune_food_monthly.csv`, `commune_monthly.csv` and `evi_timeseries/`.

## Setup

```
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Developed against Python 3.12.

## Reproducing the analysis

Run the notebooks **from inside `code/`** — they read data through relative
paths (`../data/...`), so the working directory matters. Every notebook runs
end to end on the data released here.

### `create_commune_svd_vecs.ipynb` (8 cells)

1. Loads `data/mira_cleaned_git.csv` and `data/commune_lookup.csv`.
2. Fills missing food-item values with the item mean and mean-centers the
   household x food-item matrix across all households.
3. Runs `np.linalg.svd`, reports the variance explained by the first three
   components, and plots their loadings.
4. Projects the leading component onto every household record, drops the
   undated rows, and averages to commune-month.
5. Writes `data/commune_pc1_monthly.csv` and checks it against the
   `PC1_Score` column released in `data/commune_monthly.csv` (they agree to
   3.6e-13).

### `EVI_SVD_Mean_ctrd.ipynb` (20 cells)

1. Loads the de-identified household records in `data/mira_cleaned_git.csv`
   and the commune EVI series for communes 1–6.
2. Parses the EVI dates, which are released as ISO calendar dates.
3. Fills missing values in the food-item columns with the corresponding
   commune mean, then mean-centers the matrix.
4. Runs `np.linalg.svd` on the centered matrix and projects the data onto the
   first right singular vector, producing the score `dc1` and the share of
   variance it explains.
5. Aggregates `dc1` to monthly commune means and plots it against commune EVI,
   with OLS fits and month-fixed-effects models, for **communes 1–6**.
6. Collects the slopes, p-values and R² of both specifications into one
   summary table.

### `svd_ndvi_commune_mean_centering.ipynb` (7 cells)

1. Loads the per-commune NDVI series in `data/ndvi_timeseries/` and joins them
   to the DC1 scores in `data/commune_monthly.csv`.
2. Reports the Pearson correlation between NDVI and DC1 for each commune
   (r = 0.567 to 0.758 across communes 1–6).
3. Plots NDVI against DC1 on twin axes, one figure per commune, and assembles
   the 3x2 panel figure `ndvi_vs_dc1_3x2_panel.pdf` with panels (a)–(f) in
   commune order 1–6.

### `dc1_commune_models.ipynb` (10 cells)

Fits **six OLS models**, one per commune, regressing `PC1_Score` on `ndvi`
from `data/commune_monthly.csv`, with confidence intervals — communes 1
through 6. Each commune also gets a month-fixed-effects OLS and a mixed model;
the mixed models emit convergence warnings on these short series and are not
used for the reported results.

> **To fill in:** which manuscript figure or table each notebook produces. This
> mapping is what readers and reviewers use most; without it they have to guess.

Notebook outputs are stripped from version control, so committed notebooks show
code only. Run each top to bottom in a fresh kernel to regenerate results.

## Data availability and restrictions

**No direct identifiers are distributed.** The survey's household key and the
household GPS coordinates are directly identifying in this rural setting. They
appear in the internal extract and in no file in this repository, which is
excluded from version control.

The survey data is released in two forms:

- `data/mira_cleaned_git.csv` — per-household records carrying a pseudonymous
  `ID` (1–547) in place of the survey key, no coordinates, and commune as the
  only geography.
- `data/commune_food_monthly.csv` — mean consumption per commune-month across 249
  commune-months, with commune-months of fewer than 10 households suppressed.
  Released cells draw on 14 to 145 households (median 82).

Commune-monthly PC1 scores are provided in `data/commune_monthly.csv`, so
`dc1_commune_models.ipynb` and `svd_ndvi_commune_mean_centering.ipynb`
reproduce the modeling results and figures without re-running the
decomposition. `create_commune_svd_vecs.ipynb` and `EVI_SVD_Mean_ctrd.ipynb`
both run the decomposition itself on `data/mira_cleaned_git.csv`, which holds
the same household x food-item matrix as the internal extract in de-identified
form.

> **To fill in:** the contact and procedure for researchers requesting access to
> the household-level data.

## License

- **Code** (`code/`) — MIT License, see [`LICENSE`](LICENSE)
- **Data** (`data/`) — Creative Commons Attribution 4.0 International (CC BY 4.0),
  see [`data/LICENSE`](data/LICENSE)

The administrative boundary shapefiles in `data/boundaries/` are third-party
data from BNGRC and OCHA, redistributed under their original terms. They are
**not** covered by the CC BY 4.0 grant above.

## Citation

See `CITATION.cff`.
