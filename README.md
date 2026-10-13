# ReproducibleSDOH-data

Pre-computed neighborhood socioeconomic indices for the United States, built from the US Census Bureau's American Community Survey (ACS) 5-year estimates.

This repository holds **data only**. The files are attached to its [Releases](https://github.com/HanLabCollaboration/ReproducibleSDOH-data/releases); the code that produced them is in [ReproducibleSDOH](https://github.com/HanLabCollaboration/ReproducibleSDOH).

## What is published

| Release | Index | Geographies | Scopes | ACS 5-year | Files |
|---|---|---|---|---|---|
| `yost-v2026.10` | Yost Index | tract, block group | national, state | 2011–2024 | 52 |
| `ndi-v2026.10` | Neighborhood Deprivation Index: Messer and Powell-Wiley | tract, block group (Messer); tract (Powell-Wiley) | national, state | 2011–2024 | 80 |
| `ice-v2026.10` | Index of Concentration at the Extremes (five measures) | tract, block group | — | 2011–2024 | 26 |

- **Years** name the last year of the ACS 5-year period: `2022` is the 2018–2022 ACS.
- **Block group** files start in 2013.
- **Coverage** is the 50 states and the District of Columbia.
- **Scope** is the set of areas an index is computed against. `national` compares every area with the whole country; `state` computes the index separately within each state, so values are only comparable inside a state. ICE has no scope: each value depends only on the area itself.

Each file has two versions of every score, side by side:

- **Raw:** computed from the published ACS estimates as they are. Areas with a missing component have no score.
- **Best:** with missing components filled in from neighboring areas (and, for Yost, with small-area estimates stabilized first). Far fewer areas are missing.

## File names

| Index | Pattern | Example |
|---|---|---|
| Yost | `yost_{geo}_{scope}_{year}.parquet` | `yost_tract_national_2022.parquet` |
| NDI, Messer | `ndi_messer_{geo}_{scope}_{year}.parquet` | `ndi_messer_block_group_state_2019.parquet` |
| NDI, Powell-Wiley | `ndi_powellwiley_tract_{scope}_{year}.parquet` | `ndi_powellwiley_tract_national_2015.parquet` |
| ICE | `ice_{geo}_{year}.parquet` | `ice_block_group_2021.parquet` |

`{geo}` is `tract` or `block_group`; `{scope}` is `national` or `state`.

A file's address is:

```
https://github.com/HanLabCollaboration/ReproducibleSDOH-data/releases/download/{release}/{file}
```

## What is in a file

One row per area: `GEOID` and the score columns, nothing else. The columns of every file are listed in [SCHEMA.md](SCHEMA.md).

| Index | Raw column | Best column | Direction |
|---|---|---|---|
| Yost | `Yost` | `YostStabilizedImputed` | Higher = higher socioeconomic status |
| NDI (Messer, Powell-Wiley) | `NDI` | `NDIImputed` | Higher = more deprived |
| ICE | `ICE_inc`, `ICE_edu`, `ICE_rewb`, `ICE_wbinc`, `ICE_wpcinc` | the same names with `Imputed` added | −1 = most deprived, +1 = most privileged |

Not in the files: area names, population, boundaries, and groupings such as quintiles. Join them on `GEOID` from the Census Bureau or any other source.

## Reading the files

The files are [Parquet](https://parquet.apache.org/). Download a file, then read it.

**R**

```r
url  <- "https://github.com/HanLabCollaboration/ReproducibleSDOH-data/releases/download/yost-v2026.10/yost_tract_national_2022.parquet"
file <- file.path(tempdir(), basename(url))
download.file(url, file, mode = "wb")

yost <- nanoparquet::read_parquet(file)   # or arrow::read_parquet(file)
```

**Python**

```python
import urllib.request
import pandas as pd

url = "https://github.com/HanLabCollaboration/ReproducibleSDOH-data/releases/download/yost-v2026.10/yost_tract_national_2022.parquet"
urllib.request.urlretrieve(url, "yost_tract_national_2022.parquet")

yost = pd.read_parquet("yost_tract_national_2022.parquet")   # or polars.read_parquet(...)
```

## Things to know

- **`GEOID` is text.** It has leading zeros (Alabama starts with `01`). Keep it as text; spreadsheet programs remove the zeros if they treat it as a number.
- **The state is the first two digits of `GEOID`**, and the county the first five.
- **Area definitions change in 2020.** Files for 2011–2019 use 2010 census tracts and block groups; files from 2020 on use the 2020 definitions. The two sets of `GEOID`s do not line up one to one.
- **Raw scores are missing more often in recent years.** The ACS data itself has more missing estimates for small areas in recent years, so more areas lack a component. For example, the raw Yost score is missing for about 20% of block groups in 2013 and about 40% in 2024. The best version is missing for about 1% (2013) to 2% (2024).
- **Areas with no population** have no score in either version.

## Where the files come from

Every file was computed on Stanford's Sherlock cluster with the pipeline in [ReproducibleSDOH](https://github.com/HanLabCollaboration/ReproducibleSDOH), using these packages at fixed commits:

| Release | Package | Commit |
|---|---|---|
| `yost-v2026.10` | [ReproducibleYostIndex](https://github.com/HanLabCollaboration/ReproducibleYostIndex) | `d537644` |
| `ndi-v2026.10`, `ice-v2026.10` | [ReproducibleNDI](https://github.com/HanLabCollaboration/ReproducibleNDI) | `06b5efb` |

Each file also carries this information inside it, as Parquet key–value metadata: index, geography, scope, ACS year, package version and commit, pipeline commit, R version, and the date the ACS data was downloaded. See [SCHEMA.md](SCHEMA.md).

To compute an index yourself, with other options or for other geographies, use the packages above.

## License

MIT © HanLabCollaboration (see [LICENSE](LICENSE)). The underlying ACS data is published by the US Census Bureau.
