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
  commune_food_monthly.csv      mean food consumption by commune-month
  evi_timeseries/               commune-mean EVI series, c1-c8
  commune_vecs/                 monthly NDVI + PC1 score per commune
  boundaries/                   Madagascar admin boundaries, ADM0-ADM4
  README.md                     full data dictionary
code/
  EVI_SVD_Mean_ctrd.ipynb       SVD of consumption data; DC1 vs. EVI
  dc1_commune_models.ipynb      per-commune OLS models, NDVI vs. PC1
requirements.txt
CITATION.cff
```

## Study communes

| Code | Commune | EVI series | NDVI/PC1 vectors |
|---|---|---|---|
| 1 | Tranovaho | yes | yes |
| 2 | Marolinta | yes | yes |
| 3 | — | yes | no |
| 4 | — | yes | no |
| 5 | Anjampaly | yes | yes |
| 6 | Moravato | yes | yes |
| 7 | Antaritari | yes | yes |
| 8 | Imongy | yes | yes |

Communes 3 and 4 appear in the EVI time series and in the SVD notebook, but have small sample sizes and actually fall into the boundaries of
Commune 5.

>  **Note the two numbering schemes.** `commune_food_monthly.csv` uses commune codes
> **1-6**, while the EVI series and `commune_vecs` files use **1-8**. The table
> above describes the 1-8 scheme. Confirm how survey codes 1-6 map onto it before
> a reader tries to join the two.

## Setup

```
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Developed against Python 3.12.

## Reproducing the analysis

Run both notebooks **from inside `code/`** — they read data through relative
paths (`../data/...`), so the working directory matters.

### `EVI_SVD_Mean_ctrd.ipynb` (14 cells)

1. Loads the household-level survey extract (**not distributed** — see below)
   and the commune EVI series `c1`–`c6`.
2. Converts EVI timestamps from MATLAB serial date numbers to datetimes.
3. Fills missing values in the food-item columns with the corresponding
   commune mean, then mean-centers the matrix.
4. Runs `np.linalg.svd` on the centered matrix and projects the data onto the
   first right singular vector, producing the score `dc1` and the share of
   variance it explains.
5. Aggregates `dc1` to monthly commune means and plots it against commune EVI,
   with OLS fits, for **communes 1–4**.

### `dc1_commune_models.ipynb` (10 cells)

Fits **six OLS models**, one per commune, regressing `PC1_Score` on
`mean_d1_ndvi` from the `commune_vecs` files, with confidence intervals —
communes 1, 2, 5, 6, 7, and 8.

> **To fill in:** which manuscript figure or table each notebook produces. This
> mapping is what readers and reviewers use most; without it they have to guess.

Notebook outputs are stripped from version control, so committed notebooks show
code only. Run each top to bottom in a fresh kernel to regenerate results.

## Data availability and restrictions

**All data in this repository is at the commune level or coarser. No
household-level records are distributed.**

The survey data is released as `data/commune_food_monthly.csv`: mean consumption
per commune-month across 249 commune-months, with commune-months of fewer than
10 households suppressed. Released cells draw on 14 to 145 households (median 82).

The underlying household-level extract contains a household key and GPS
coordinates, which are directly identifying in this rural setting. It is not
included here under any filename, and is excluded from version control.

Because the decomposition runs on a household x food-item matrix,
`EVI_SVD_Mean_ctrd.ipynb` cannot be re-run from the released aggregate.
Its published outputs — commune-monthly PC1 scores — are provided in
`data/commune_vecs/`, so `dc1_commune_models.ipynb` reproduces the modeling
results in full.

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
