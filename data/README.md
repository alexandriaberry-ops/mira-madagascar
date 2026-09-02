# Data

## `mira_cleaned_git.csv`

Per-household food consumption records, de-identified. 22,934 rows x 36 columns,
547 households, 2018-2022.

| Column | Description |
|---|---|
| `ID` | Pseudonymous household identifier, 1-547. Not the survey's household key. |
| `year`, `month`, `day` | Observation date |
| *food columns* | Per-item consumption, same 31 items as `commune_food_monthly.csv` |
| `commune` | Commune code (1-6) |

The survey key and the household GPS coordinates present in the internal extract
are both removed. `commune` uses the same 1-6 numbering as every other file here;
it was checked against `commune_food_monthly.csv` commune by commune before
release.

**1,781 rows (7.8%) have blank `year`, `month` and `day`** — see the data quality
notes below.

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

## `commune_monthly.csv`

Commune-mean NDVI paired with the first principal component score of household
food consumption, by month. 256 rows.

| Column | Description |
|---|---|
| `commune` | Commune code (1-6) |
| `YearMonth` | Month, `YYYY-MM` |
| `ndvi` | Commune-mean NDVI for that month, weighted by household locations (see Methods) |
| `PC1_Score` | Mean first principal component score across that commune's households |

Input to `code/ndvi/dc1_commune_models.ipynb` and
`code/ndvi/svd_ndvi_commune_mean_centering.ipynb`. Covers 2018-07 through 2022-02.
The `PC1_Score` column is reproduced by `code/ndvi/create_commune_svd_vecs.ipynb`.
Joins to `commune_food_monthly.csv` on `commune` plus month.

## `food_security_monthly.csv`

The four commonly used household food security indices, averaged to
**commune x month**. 247 rows.

| Column | Description |
|---|---|
| `commune` | Commune code (1-6) |
| `YearMonth` | Month, `YYYY-MM` |
| `n_households` | Households contributing to the cell |
| `FCS` | Food Consumption Score |
| `HDDS_24hr` | Household Dietary Diversity Score, 24-hour recall |
| `HHS` | Household Hunger Scale |
| `rCSI1` | Reduced Coping Strategies Index |

Derived from the internal household-level extract: households answering fewer
than 70% of survey waves were dropped, records with no date were dropped, and
the remaining records were averaged within each commune-month. Input to
`code/ndvi/food_security_indices_v_ndvi.ipynb`, which reproduces the reported
correlations against the `ndvi` column of `commune_monthly.csv`.

**Four cells rest on fewer than 10 households** -- commune 5 in 2018-07 (n = 2),
2018-12 (n = 4) and 2021-04 (n = 7), and commune 6 in 2018-07 (n = 3). Unlike
`commune_food_monthly.csv`, which suppresses cells below that threshold, these
are released so that the reported correlations reproduce exactly; the
`n_households` column identifies them. A mean over very few households
approaches individual disclosure, and those four cells should be treated
accordingly. All other cells draw on 14 to 145 households.

The Household Hunger Scale is reported as zero by most households in most
months, so its commune means are zero for long stretches, particularly from
2021 onward.

## `commune_lookup.csv`

The commune numbering used throughout this repository, with each commune's name
and its code in the source shapefile.

| Column | Description |
|---|---|
| `commune` | Commune code used in every released file (1-6) |
| `commune_name` | Commune name |
| `shapefile_code` | Position in the BNGRC/OCHA ADM4 shapefile the series were extracted from |
| `note` | Aggregation notes, where they apply |

Codes 1-6 run left to right across the study area, matching the manuscript
figures. The shapefile codes are recorded only so the series can be traced back
to their source; no released file uses them.

## `evi_timeseries/commune_1_evi.csv` ... `commune_6_evi.csv`

Mean Enhanced Vegetation Index per commune over time, at the native 16-day
composite resolution. One file per commune, 542 observations each, spanning
2000-02-18 to 2023-08-29.

| Column | Description |
|---|---|
| `date` | Observation date, `YYYY-MM-DD` |
| `evi` | Commune-mean EVI for that date |

Communes 1-6 are loaded and modeled by the SVD notebook.

## `ndvi_timeseries/commune_1_ndvi.csv` ... `commune_6_ndvi.csv`

Mean Normalized Difference Vegetation Index per commune, at the native 16-day
composite resolution. One file per commune, 507 observations each, spanning
2000-02-18 to 2022-02-18.

| Column | Description |
|---|---|
| `date` | Observation date, `YYYY-MM-DD` |
| `ndvi` | Commune-mean NDVI for that date |

Extracted per commune polygon from the satellite record. `code/ndvi/svd_ndvi_commune_mean_centering.ipynb`
averages them to calendar months before pairing them with DC1.

These are the unweighted polygon means for each commune. They are **not** the
same quantity as the `ndvi` column of `commune_monthly.csv`, which is the
household-location-weighted commune average described in the Methods and is the
series all reported models and correlations use. The two agree closely
(*r* = 0.998 per commune, maximum absolute difference 0.025) but are not
interchangeable; use `commune_monthly.csv` to reproduce the tables.

## `boundaries/mdg_adm_bngrc_ocha_20181031_shp/`

Madagascar administrative boundaries, ADM0 through ADM4, published by BNGRC and
OCHA (2018-10-31). Commune-level units are ADM4. Standard ESRI shapefile
components (`.shp`, `.shx`, `.dbf`, `.prj`, and index files) for each level.

Source: OCHA Humanitarian Data Exchange. Redistributed under the terms of the
original release; see the accompanying `.xml` metadata files.

## Data quality notes

- **Commune 4 (Anjampaly) mixes two aggregation footprints.** Shapefile communes 3
  (Nikoly) and 4 (Betanty / Faux Cap) were collapsed into Anjampaly during survey
  aggregation, so its food data includes their households, while its NDVI and EVI
  series cover the Anjampaly polygon alone. Treat that commune's greenness as a
  proxy for a slightly larger survey population than the polygon it is drawn from.
- **1,781 rows (7.8%) have blank `year`, `month`, and `day`**, spread across 518
  of the 547 households. These are undated observations, not duplicates, though
  they collapse to identical keys if you index on household and date. Filter or
  impute deliberately before any time-series step.
- Dated records span 2018-2022.
