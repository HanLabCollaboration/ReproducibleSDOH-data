# Schema

Schema version: **1** (releases `yost-v2026.10`, `ndi-v2026.10`, `ice-v2026.10`).

Every file is a Parquet table with one row per area, sorted by `GEOID`. `GEOID` is unique within a file and never missing. Score columns are missing (null) where the index could not be computed.

## Columns

### Yost: `yost_{geo}_{scope}_{year}.parquet`

| Column | Type | Description |
|---|---|---|
| `GEOID` | string | Census identifier: 11 digits for a tract, 12 for a block group |
| `Yost` | double | Raw Yost Index: no stabilization, no imputation |
| `YostStabilizedImputed` | double | Yost Index after stabilizing the components and imputing missing ones from neighboring areas |

Higher values mean higher socioeconomic status. Scores are centered on 10 within each scope group (the country, or each state).

### NDI: `ndi_messer_{geo}_{scope}_{year}.parquet` and `ndi_powellwiley_tract_{scope}_{year}.parquet`

| Column | Type | Description |
|---|---|---|
| `GEOID` | string | Census identifier: 11 digits for a tract, 12 for a block group |
| `NDI` | double | Raw Neighborhood Deprivation Index: no imputation |
| `NDIImputed` | double | NDI after imputing missing components from neighboring areas |

Higher values mean more deprivation. The Messer and Powell-Wiley files have the same columns but hold different indices; the method is in the file name and in the file's `series` metadata.

### ICE: `ice_{geo}_{year}.parquet`

| Column | Type | Description |
|---|---|---|
| `GEOID` | string | Census identifier: 11 digits for a tract, 12 for a block group |
| `ICE_inc` | double | Income: high-income minus low-income households |
| `ICE_edu` | double | Education: bachelor's degree or more minus less than high school, age 25+ |
| `ICE_rewb` | double | Race/ethnicity: non-Hispanic White minus non-Hispanic Black |
| `ICE_wbinc` | double | Race and income: high-income non-Hispanic White minus low-income Black households |
| `ICE_wpcinc` | double | Income among non-Hispanic White households: high-income minus low-income |
| `ICE_incImputed`, `ICE_eduImputed`, `ICE_rewbImputed`, `ICE_wbincImputed`, `ICE_wpcincImputed` | double | The same five measures after imputing missing ones from neighboring areas |

Every value lies between −1 (most deprived) and +1 (most privileged).

## Values in file names

| Part | Values |
|---|---|
| `{geo}` | `tract`, `block_group` |
| `{scope}` | `national`, `state` |
| `{year}` | `2011`–`2024` for tract; `2013`–`2024` for block group. The year is the last year of the ACS 5-year period |

## Metadata inside each file

Each file carries these key–value pairs. All values are stored as text.

| Key | Example | Meaning |
|---|---|---|
| `schema_version` | `1` | Version of this schema |
| `index` | `yost`, `ndi` | Pipeline folder that produced the file |
| `series` | `messer`, `powell_wiley`, `ice` | Which index of the NDI package (NDI and ICE files only) |
| `geo` | `tract`, `block group` | Geography |
| `scope` | `national`, `state`, `none` | Reference set (`none` for ICE) |
| `acs_year` | `2022` | Last year of the ACS 5-year period |
| `acs_survey` | `acs5` | ACS product |
| `release_tag` | `yost-v2026.10` | Release the file belongs to |
| `package` | `ReproducibleYostIndex` | R package that computed the index |
| `package_version` | `0.0.0.9000` | Its version |
| `package_git_sha` | `d5376443…` | Its exact commit |
| `pipeline_git_sha` | `640f35f3…` | Commit of the ReproducibleSDOH pipeline |
| `r_version` | `4.4.2` | R version |
| `tidycensus_version` | `1.8.1` | Version of the package used to download ACS data |
| `fetched_at` | `2026-10-13T01:14:11Z` | When the task ran (UTC) |
| `geoinfo_release` | *(empty)* | Reserved for a future release of area names and population |

To read the metadata:

```r
nanoparquet::read_parquet_metadata(file)$file_meta_data$key_value_metadata[[1]]
```

```python
import pyarrow.parquet as pq
pq.read_metadata(file).metadata
```

## Checksums

Each release includes a `SHA256SUMS` file listing the SHA-256 checksum of every Parquet file in it.
