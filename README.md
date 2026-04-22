# FPFA Carbon Intensity & Macrogrid Tool

Interactive map of US grid carbon intensity and the Scope 2 reduction potential from a macrogrid scenario, focused on energy-intensive manufacturing sectors.

## How to run

```bash
# 1. Preprocess raw data (run once, or after updating any raw file)
python3 build.py

# 2. Serve from project root
python3 -m http.server

# 3. Open http://localhost:8000 in your browser
```

Dependencies: `geopandas pandas openpyxl numpy`  
Install with: `pip install geopandas pandas openpyxl numpy`

## Data sources

| File | Source | URL |
|------|--------|-----|
| `data/raw/egrid2023_subregions/` | EPA eGRID 2023 Subregion Shapefiles | https://www.epa.gov/egrid/egrid-related-files |
| `data/raw/egrid2023_rates.csv` | EPA eGRID 2023 Summary Tables (Table 1) | https://www.epa.gov/egrid/download-data |
| `data/raw/eGRID2023_Data.xlsx` | EPA eGRID 2023 Data File (net generation from Table 2) | https://www.epa.gov/egrid/download-data |
| `data/raw/mecs_electricity.xlsx` | EIA MECS 2022, Table 7.6 | https://www.eia.gov/consumption/manufacturing/data/2022/ |
| `data/raw/tri_facilities.csv` | EPA TRI 2023 Facility-Level Data | https://www.epa.gov/toxics-release-inventory-tri-program/tri-basic-plus-data-files-calendar-years-1987-present |

## Methodology

**Scope 2 current emissions** are estimated by distributing EIA MECS 2022 electricity consumption (by sector and Census region) to individual eGRID 2023 subregions, weighted by each subregion's share of net generation within its Census region. Emissions are computed using subregion annual CO₂ output emission rates (lb/MWh) from eGRID 2023.

**Macrogrid scenario** replaces each subregion's emission rate with the national demand-weighted average rate (767.2 lb/MWh). The difference is the reduction potential.

**Sectors covered:** Iron & Steel (NAICS 3311–3312), Aluminum (3313–3314), Cement (32731), Glass (32721–32722), Fertilizer (32531), Hydrogen/Industrial Gases (32512), Solar manufacturing (33441).

**Facility locations** are from EPA TRI 2023; deduplicated by name + coordinates.

## Limitations

- The macrogrid counterfactual is a static rate substitution. It excludes transmission constraints, grid stability requirements, and the induced renewables buildout that a physical macrogrid would enable — likely **understating** the true benefit.
- MECS electricity is 2022 data; eGRID rates are 2023. The one-year mismatch is minor but not zero.
- MECS suppresses values (reported as `Q`, `W`, `D`, `*`) for confidentiality; those cells are excluded from calculations.
- TRI reporting thresholds exclude smaller facilities. Solar manufacturing in particular may be significantly undercounted.
- Alaska (AKGD, AKMS) and Puerto Rico (PRMS) are excluded from the macrogrid calculation — no MECS Census region mapping.
