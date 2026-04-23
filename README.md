# Grid's Impact on EITE Industry Carbon Intensity

Interactive map of US grid carbon intensity and the Scope 2 emission reduction potential from improved interregional transmission, focused on energy-intensive and trade-exposed (EITE) manufacturing sectors.

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

---

## Data sources

| File | Source | Notes |
|------|--------|-------|
| `data/raw/egrid2023_subregions/` | [EPA eGRID 2023 Subregion Shapefiles](https://www.epa.gov/egrid/detailed-data) | Boundary geometries for 26 subregions |
| `data/raw/egrid2023_rates.csv` | [EPA eGRID 2023 Summary Tables (Table 1)](https://www.epa.gov/egrid/detailed-data) | Annual CO₂ output emission rates and net generation by subregion |
| `data/raw/eGRID2023_Data.xlsx` | [EPA eGRID 2023 Data File (Table 2)](https://www.epa.gov/egrid/detailed-data) | Net generation used to compute subregion shares |
| `data/raw/mecs_electricity.xlsx` | [EIA MECS 2022, Table 7.6](https://www.eia.gov/consumption/manufacturing/data/2022/) | Electricity consumption by sector and Census region |
| `data/raw/tri_facilities.csv` | [EPA TRI 2023 Facility-Level Data](https://www.epa.gov/toxics-release-inventory-tri-program/tri-basic-data-files-calendar-years-1987-present) | Facility name, location, NAICS code |

---

## Methodology

### Sectors covered

Seven EITE manufacturing sectors are modeled, identified by NAICS code:

| Sector | NAICS |
|--------|-------|
| Iron & Steel | 3311, 3312 |
| Aluminum | 3313, 3314 |
| Cement | 32731 |
| Glass | 32721, 32722 |
| Fertilizer (nitrogenous) | 32531 |
| Hydrogen / Industrial Gases | 32512 |
| Solar panel manufacturing | 33441 |

### Step 1 — Current Scope 2 emissions

[EIA MECS 2022 Table 7.6](https://www.eia.gov/consumption/manufacturing/data/2022/) reports electricity consumption (in million kWh) by sector and Census region (Northeast, Midwest, South, West). These four Census regions do not align with the 26 [eGRID 2023](https://www.epa.gov/egrid/detailed-data) subregions, so consumption is disaggregated using each subregion's share of its Census region's total net generation:

```
share(subregion) = net_gen(subregion) / Σ net_gen(subregions in same Census region)

elec_MWh(sector, subregion) = MECS_mkWh(sector, Census region) × 1,000,000 × share(subregion)

Scope2_current(sector, subregion) = elec_MWh × CO₂_rate(subregion) / 2,204.6
```

The division by 2,204.6 converts pounds of CO₂ to metric tons. Displayed emission rates are converted from lb CO₂/MWh to **kg CO₂/MWh** (÷ 2.2046) for readability.

### Step 2 — National demand-weighted average rate

The reference rate used in all scenarios is the generation-weighted national average across all eGRID subregions:

```
R_national = Σ (CO₂_rate × net_gen) / Σ net_gen
```

This is approximately 363 kg CO₂/MWh (800 lb/MWh) for the 2023 eGRID data.

### Step 3 — Three transmission scenarios

Each scenario estimates what Scope 2 emissions would be if interregional transmission allowed cleaner electricity to displace higher-emission generation. The reduction potential for each cell is:

```
Reduction = (Scope2_current − Scope2_scenario)

where Scope2_scenario = elec_MWh × R_effective / 2,204.6
```

`R_effective` depends on the scenario:

#### Scenario 1: Full Macrogrid

All subregions adopt the national demand-weighted average rate. Hawaii islands (HIMS, HIOA) are physically excluded.

```
R_effective = R_national   [for all subregions except HIMS, HIOA]
```

This is a theoretical upper bound — it assumes perfectly frictionless interregional power flow and no transmission losses.

#### Scenario 2: NIETC Corridors (~40% equalization)

Models the five [National Interest Electric Transmission Corridors](https://www.energy.gov/oe/national-interest-electric-transmission-corridor-designation-process) designated by the U.S. Department of Energy in 2024. These corridors are intended to facilitate interregional power transfer across specific interfaces. Corridor regions are assumed to close approximately 40% of the gap between their current rate and the national average; regions outside the corridors see no change. Texas (ERCT) and Hawaii are excluded.

Corridor regions: NEWE, NYCW, NYUP, NYLI, RFCE, RFCM, RFCW, MROW, MROE, SPNO, CAMX, NWPP, AZNM, RMPA, SRVC, SRSO.

```
R_effective = R_regional − 0.40 × (R_regional − R_national)   [corridor regions]
R_effective = R_regional                                         [all others]
```

The 40% factor is an approximation based on the scope and projected capacity of the designated corridors. It should be updated as DOE publishes project-level capacity and flow estimates.

#### Scenario 3: NERC ITCS (~55% equalization)

Based on NERC's [Interregional Transfer Capability Study (ITCS)](https://www.nerc.com/globalassets/initiatives/itcs/itcs_final_report.pdf), which analyzed transfer capability across seven interregional interfaces and found substantial room for improvement. All subregions in the Eastern and Western interconnections are assumed to close 55% of the gap. Texas (ERCT) and Hawaii remain isolated.

```
R_effective = R_regional − 0.55 × (R_regional − R_national)   [all except ERCT, HIMS, HIOA]
```

The 55% factor is derived from the ITCS finding that a large share of the theoretical interregional transfer benefit is technically achievable with identified infrastructure investments.

### NERC Regional Entities

Subregions are grouped under six NERC Regional Entities for reference:

| Abbr | Full name | Subregions |
|------|-----------|------------|
| MRO | Midwest Reliability Organization | MROE, MROW, SPNO |
| NPCC | Northeast Power Coordinating Council | NEWE, NYCW, NYUP, NYLI |
| RF | ReliabilityFirst | RFCE, RFCM, RFCW |
| SERC | Southeast Regional Council | SPSO, SRMV, SRMW, SRSO, SRTV, SRVC, FRCC |
| Texas RE | Texas Reliability Entity | ERCT |
| WECC | Western Electricity Coordinating Council | AZNM, CAMX, NWPP, RMPA, HIMS, HIOA |

### Facility locations

[EPA TRI 2023 facility-level data](https://www.epa.gov/toxics-release-inventory-tri-program/tri-basic-data-files-calendar-years-1987-present) is filtered to the seven NAICS codes above and deduplicated by name and coordinates. Facilities are assigned to eGRID subregions using point-in-polygon matching. Facility counts are shown in the subregion detail table for context but are not used in emissions calculations.

---

## Caveats and limitations

### Methodology caveats

**Static rate substitution.** All three scenarios are static counterfactuals: they substitute a different emission rate for each subregion's electricity, without modeling how generation dispatch, grid stability, or investment would respond. A real macrogrid or expanded transmission network would induce additional renewables buildout and shift dispatch toward cleaner sources — effects that are not captured here. The scenarios almost certainly **understate** the full emissions benefit.

**Net generation share assumption.** MECS electricity consumption is reported at the Census region level (four regions). Disaggregation to subregions uses each subregion's share of its Census region's net generation. This assumes that industrial electricity demand within a Census region mirrors the generation mix — a simplification that ignores spatial variation in industrial siting relative to generation infrastructure.

**Scenario factors are approximations.** The 40% equalization factor for NIET corridors and the 55% factor for NERC ITCS are informed estimates, not values derived directly from those reports' published flow or capacity data. They should be treated as order-of-magnitude illustrations pending more granular modeling.

**Regions below the national average.** Under any equalization scenario, subregions whose current rate is below the national average would see their effective rate increase (they "export" clean electricity to dirtier regions). This is economically realistic in a market-clearing macrogrid but the static model does not capture the investment or pricing dynamics that would accompany it.

### Data caveats

**Data year mismatch.** MECS electricity consumption is from 2022; eGRID emission rates are from 2023. The one-year gap is minor but not zero, particularly for subregions that saw large changes in generation mix.

**MECS data suppression.** EIA suppresses values (reported as `Q`, `W`, `D`, or `*`) for confidentiality when cell counts are small. Suppressed cells are excluded from calculations, which means some sector–region combinations are missing and total Scope 2 is a lower bound.

**TRI reporting thresholds.** TRI requires reporting only above certain chemical release or manufacturing thresholds. Smaller facilities, and sectors with limited reporting requirements, are undercounted. Solar panel manufacturing in particular is likely significantly underrepresented.

**Alaska and Puerto Rico excluded.** AKGD, AKMS, and PRMS have no Census region mapping in MECS and are excluded from all Scope 2 calculations.

**eGRID annual averages.** eGRID rates are annual averages. They do not capture seasonal or diurnal variation in grid carbon intensity, which affects the real-time emissions benefit of demand flexibility and storage.
