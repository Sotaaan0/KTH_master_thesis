# Forecasting new vehicle registrations and CO₂ emissions for Swedish municipalities

Code and data for the master's thesis *Forecasting new vehicle registrations
and CO₂ emissions for Swedish municipalities* (KTH Royal Institute of
Technology). The project forecasts new passenger-car registrations across five
fuel types — petrol, diesel, electricity, electric-hybrid, plug-in hybrid —
and the resulting fleet CO₂ emissions for all 290 Swedish *kommuner* through
2040, using log-log regressions on socioeconomic predictors together with the
SKR nine-group municipality classification.

**Author:** Sota Yoshimoto

---

## Repository layout

```
.
├── 1_API.ipynb        # data preparation: pulls SCB/Trafikanalys/SKR sources,
│                      # builds the master panel, writes output/df.csv
├── 2_main.ipynb       # regression, validation, forecast, CO₂ projection
├── data/              # raw input files (mostly bundled, see Data sources below)
│   ├── Kommun_bidding.csv
│   ├── vkt_data_2024.xlsx
│   ├── EEA_2023_SE_data.csv     # NOT bundled — ~70 MB, download manually (see below)
│   └── data_new/
│       ├── df_registration.xlsx
│       └── df_onwer_type.xlsx
├── output/            # created at runtime; receives df.csv, plots, .tex table
└── README.md
```

## Running the notebooks

Run `1_API.ipynb` first (it writes `output/df.csv`), then `2_main.ipynb`.

### Before running: download the EEA emissions file

The EEA dataset used in `2_main.ipynb` (~70 MB) is **not bundled** in this
repository because of its size. Download it manually before running the
second notebook:

1. Go to the EEA Data Hub page for *CO₂ emissions from new passenger cars*:
   <https://www.eea.europa.eu/en/datahub/datahubitem-view/5d252092-d328-40d8-bca2-c0734bd6143b>
2. Download the dataset for **Sweden, year 2023** (the EEA portal lets you
   filter by country and year before exporting).
3. Save it as `data/EEA_2023_SE_data.csv`.

The notebook reads only four columns from this file: `Ft` (fuel type),
`Fm` (fuel mode), `Ewltp (g/km)` (WLTP CO₂), and `year`. Any export that
includes those columns is fine.

### Requirements

Python 3.10+ and:

```
pandas
numpy
requests
statsmodels
scikit-learn
matplotlib
seaborn
scipy
openpyxl       # for reading .xlsx files
jupyter
```

Quick install:

```bash
pip install pandas numpy requests statsmodels scikit-learn matplotlib seaborn scipy openpyxl jupyter
```

### Notes

- `1_API.ipynb` hits the SCB JSON-stat API live for income, education,
  urbanization, household, density, and electricity-price series. The cells
  are stable but if SCB's endpoints change you may need to update the URLs.
- `2_main.ipynb` reads only local files. No network access required.

## Outputs

`output/df.csv` is the canonical municipality-year panel produced by
`1_API.ipynb`. `2_main.ipynb` additionally writes:

- `output/residual_before_after_log.png` — residual diagnostics (paper Fig 3.2)
- `output/residual_before_after_log_1.png`, `_2.png` — split versions
- `output/emissions_9municipalities.png` — 3×3 stacked emissions plot
  (paper Fig 4.3)
- `output/table9_representative_municipalities.tex` — standalone LaTeX table
  (paper Table 4.7)

## Code ↔ paper model mapping

The four regression result dictionaries used in `2_main.ipynb` correspond to
the paper's models as follows. (The code went through several drafts in which
the numbering shifted, so this mapping is the authoritative one.)

| Variable in code | Paper |
| --- | --- |
| `results_summary_lin` | Linear baseline — diagnostic only (not in paper) |
| `results_summary_m1` | **Model 1** — log-log, 2023 cross-section |
| `results_summary_m2` | **Model 2** — log-log, 2019–2023 panel, no trend |
| `results_summary_m3` | **Model 3** — log-log + quadratic time trend, 2019–2023 |

The same numbering carries through to the validation tables: `df_validation_2024`
(Model 1 on 2024), `df_val_m2`, `df_val_m3`.

## Data sources

Most files under `data/` are bundled in this repository for reproducibility.
The EEA file is the exception — it must be downloaded manually (see *Before
running* above). Original sources and licensing:

| File | Source | License / terms |
| --- | --- | --- |
| `data_new/df_registration.xlsx` | Trafikanalys (registration series) + SKR (municipality classification, `kommun_group` sheet) | Trafikanalys: free redistribution with attribution. SKR: free use with attribution |
| `data_new/df_onwer_type.xlsx` | Trafikanalys (vehicles by owner type) | Free redistribution with attribution |
| `vkt_data_2024.xlsx` | Trafikanalys 2024 national report — vehicle kilometres travelled by fuel | Free redistribution with attribution |
| `EEA_2023_SE_data.csv` *(not bundled, ~70 MB)* | European Environment Agency — CO₂ emissions from new passenger cars (Sweden, 2023) | EEA open data — download manually |
| `Kommun_bidding.csv` | Author-compiled mapping from `Kommun-kod` to Sweden's four electricity bidding zones (SE1–SE4) | Original work |

The notebook also fetches the following series live from SCB:

- `UF0506B/Utbildning` — share of population with bachelor's degree or higher
- `HE0110G/TabVX4bDispInkN` — median household disposable income
- `MI0810A/TatortGrad` — degree of urbanization
- `BE0101S/HushallT03` — total households
- `BE0101C/BefArealTathetKon` — population density
- `EN0301A/SSDManadElhandelpris` — monthly electricity prices

SCB data is published under the CC0 license.

**Required attribution when redistributing the bundled Trafikanalys data:**
`Source: Trafikanalys`. See <https://www.trafa.se/om-oss/om-webbplatsen/> for
the full terms.

## Caveats baked into the forecast

- Predictor variables (income, education, household count, etc.) are projected
  with CAGR computed on 2019–2024; structural breaks beyond this window are
  not modelled. An average-growth alternative is available in the same code.
- The quadratic time trend in Model 3 is frozen at its 2024 value (τ = 5,
  τ² = 25) for all forecast years, preventing extrapolated explosion. This is
  discussed in paper Section 3.5.
- Vehicle kilometres travelled (VKT) is held constant per fuel type using
  the 2024 Trafikanalys figures. Travel-behaviour change associated with
  electrification is not modelled.
- Emission factors are derived from the 2023 EEA new-registration dataset
  for Sweden and held constant; cohort effects from older vehicles still in
  the fleet are not modelled.

## License

Code: MIT (or the author's preferred license — adjust before publishing).

Data: each bundled file retains its source license, summarised above.
