# Data

## `commune_food_monthly.csv`

Mean household food consumption, aggregated to **commune x month**. 249 rows x 35
columns. This is the survey data released with this repository; no household-level
records are distributed.

| Column | Description |
|---|---|
| `commune` | Commune code (1-6) |
| `year`, `month` | Observation month |
| `n_households` | Households contributing to the cell |
| *food columns* | Mean per-item consumption across those households (Maize, Sorghum, Millet, Rice, Cassava Root, Cassava Leaves, Sweet Potato, Peanuts, Cowpeas, Beans, cactus, vegetables, fruit, beef, milk products, fish products, eggs, sugar, and others) |

Derived from the internal household-level extract as follows:

1. Records with no `year`/`month`/`day` were dropped (1,781 of 22,934 rows, 7.8%).
2. Remaining records were averaged within each commune-month.
3. **Cells with fewer than 10 contributing households were suppressed** — 7 of 256
   commune-months, including one built from a single household. A mean over very
   few households approaches individual disclosure.

Released cells contain between 14 and 145 households, median 82.

The household-level extract itself contains a household key and GPS coordinates
and is **not** part of this repository under any filename.

## `evi_timeseries/c1_evi_ts.csv` ... `c8_evi_ts.csv`

Mean Enhanced Vegetation Index per commune over time. One file per commune.

| Column | Description |
|---|---|
| `evi_time` | Observation date, serial date number |
| `cN_mean_evi` | Commune-mean EVI for that date |

Communes 1-6 are loaded by the SVD notebook, which models communes 1-4.
Communes 7 and 8 are included for completeness.

## `commune_vecs/commune_N_vecs.csv`

Per-commune monthly series pairing NDVI with the first principal component score.
Inputs to the commune-level models.

| Column | Description |
|---|---|
| `YearMonth` | Month, `YYYY-MM` |
| `mean_d1_ndvi` | Commune-mean NDVI |
| `PC1_Score` | First principal component score |

Present for communes 1, 2, 5, 6, 7, 8 -- named, in order: Tranovaho, Marolinta,
Anjampaly, Moravato, Antaritari, Imongy. No files exist for communes 3 and 4,
so those two are absent from the OLS models.

## `boundaries/mdg_adm_bngrc_ocha_20181031_shp/`

Madagascar administrative boundaries, ADM0 through ADM4, published by BNGRC and
OCHA (2018-10-31). Commune-level units are ADM4. Standard ESRI shapefile
components (`.shp`, `.shx`, `.dbf`, `.prj`, and index files) for each level.

Source: OCHA Humanitarian Data Exchange. Redistributed under the terms of the
original release; see the accompanying `.xml` metadata files.

## Data quality notes

- **Commune codes differ between files.** `commune_food_monthly.csv` uses codes 1-6;
  the EVI series and `commune_vecs` files use 1-8. Do not join on commune code
  without first confirming how the two schemes correspond.
- **1,781 rows (7.8%) have blank `year`, `month`, and `day`**, spread across 518
  of the 547 households. These are undated observations, not duplicates, though
  they collapse to identical keys if you index on household and date. Filter or
  impute deliberately before any time-series step.
- Dated records span 2018-2022.
