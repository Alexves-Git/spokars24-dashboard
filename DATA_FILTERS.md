# SPOKARS-24 Dashboard — Data Filters and Noise Documentation

**Internal reference only. Do not distribute to exercise teams.**

This document describes all noise filters, reporting adjustments, and data transformations applied to the SPOKARS-24 Surveillance Dashboard. These details are not disclosed to the teams participating in the C-CHA tabletop exercise.

---

## Data Sources

The dashboard integrates data from two simulation frameworks:

- **GLEAM** (Global Epidemic and Mobility Model): 3,253 subpopulations, 232 countries, states and territories, 36 weeks of simulation. Source file: `master_world-cities.csv` (~787K rows). Used for global death and case data.
- **EpiHiper**: Agent-based model used for four locations (California, Virginia, France, United Kingdom), initialized with GLEAM importations. EpiHiper output replaces GLEAM data entirely for these locations.

### Dashboard Time Window

Epi weeks 202446 through 202502 (November 16, 2024 – January 11, 2025), 9 weeks total.

### Death Field

`Deaths_total_delayed` (not `Deaths_total`) — incorporates realistic reporting delays from the GLEAM simulation.

### Cases Field

`Infectious_symp_total_detected_delayed` from GLEAM world cases file (`master_world_cases_noUS-AUS_csv.gz`), aggregated from daily to epi weeks.

---

## Deaths — Noise and Filters

### Vietnam

- **30% reporting probability**: Each weekly death is independently sampled with a binomial distribution at p = 0.30. This simulates severe underreporting.

### All other GLEAM countries (except EpiHiper substitutions and Vietnam)

- **±10% uniform noise**: Each weekly death count is multiplied by a uniform random factor in [0.9, 1.1], then rounded.

### Korea, Japan, Thailand

- **Week 202502 (Jan 11)**: Death data set to `null`. Simulates reporting delays of up to two weeks in countries with overwhelmed testing capacity.

### EpiHiper substitutions (no additional noise)

The following locations use raw EpiHiper output, replacing GLEAM data entirely within the dashboard time window:

- **California**: Deaths and cases from EpiHiper weekly files.
- **Virginia**: Cases only from EpiHiper. Deaths set to zero (no EpiHiper death file provided).
- **France**: Deaths and cases from EpiHiper weekly files.
- **United Kingdom**: Deaths and cases from EpiHiper weekly files.

---

## Age Breakdown (Deaths)

- Original age groups collapsed to three categories: **Under 65**, **65+**, **Unknown**.
- Approximately **30%** of deaths are randomly reclassified as "Unknown" age, per country per week. This simulates incomplete demographic reporting.

---

## Cases — Noise and Filters

### Australia

- Source: `aus-country-cases.csv` (daily country-level detected cases).
- Aggregated to epi weeks. **No noise applied.**

### United States (state-level)

- Source: `us_states_cases.csv` (daily detected_delayed cases by state).
- Aggregated to epi weeks. **No noise applied.**
- For weeks ≤ 202501 (before the case notification system was fully active), weekly deaths were added to weekly cases.

### France, United Kingdom, California, Virginia

- Source: EpiHiper weekly case files.
- **No noise applied.** Raw simulation output.

### New Zealand

- Source: GLEAM detected cases.
- Aggregated to epi weeks. **No noise applied.**

### 90% Reporting Group

Each daily case count sampled with binomial distribution at p = 0.90:

- Italy
- Switzerland
- Spain
- Germany
- Ireland
- Israel
- Canada
- Singapore

### 70% Reporting Group

Each daily case count sampled with binomial distribution at p = 0.70:

- Fiji

### Japan, South Korea

- Source: GLEAM detected cases.
- **Full reporting (no noise).** Originally processed at 70% reporting probability, later revised to 100%.
- **Week 202502**: Cases set to `null`, consistent with missing death data for that week.

### 50% Reporting Group

Each daily case count sampled with binomial distribution at p = 0.50:

- Argentina
- South Africa
- Brazil

Note: These countries have minimal or zero cases in the dashboard time window due to late epidemic arrival.

### Deaths-as-Cases Rule

For **Japan, South Korea, New Zealand, and the United States** (including state-level data): in any week where deaths exceed reported cases (or cases are null but deaths exist), deaths are counted as confirmed cases. This ensures case counts are never lower than death counts.

### Importation Events as Cases

All confirmed importation events (53 total) are added as cases to their destination countries in the corresponding epi week. This affects countries that receive travel-related importations, particularly in weeks 202446–202447. Countries affected include China (+6), New Zealand (+3), Canada (+2), India (+2), USA (+2), Japan (+1), South Korea (+1), Thailand (+1), Mexico (+1), UAE (+1), Vietnam (+1), and others. Note: importation events prior to week 202446 (Oct 26 – Nov 10) fall outside the dashboard time window and are not reflected in case counts.

---

## Importation Data

### Sources

- **WHO Situation Reports #1–4** (Oct 26 – Nov 10): 27 events, all from Australia.
- **National health authority reports** (Nov 11 – Nov 23): 26 additional events from multiple sources.

### Source Countries and Detection Filters (post Nov 10)

Events from the GLEAM international importations file (`master_world_international_importations_11-10_11-23.csv`) were filtered as follows:

- **Australia**: All events included (100%).
- **New Zealand**: 50% detection probability per event (seed 42).
- **Fiji**: All events included (100%).
- **Solomon Islands**: All events included (100%).
- **Singapore**: 10% detection probability per event (seed 42).
- **Far East secondary sources** (Korea, Vietnam, Japan, Thailand, China, Taiwan, Philippines, Malaysia): 10% detection probability per event (seed 123).
- **All other sources** (UK, Spain, Austria, Switzerland, USA, Canada, France, Saudi Arabia, Samoa, Sri Lanka): 5% detection probability per event (seed 123).

### Resulting Source Attribution (53 total events)

| Source | Events | Color on map |
|---|---|---|
| Australia | 32 | Amber (#f59e0b) |
| Singapore | 6 | Purple (#a855f7) |
| Fiji | 5 | Cyan (#06b6d4) |
| South Korea | 4 | Orange (#f97316) |
| New Zealand | 3 | Green (#22c55e) |
| Vietnam | 2 | Teal (#14b8a6) |
| Solomon Islands | 1 | Pink (#ec4899) |

### Reporting Authorities (post Nov 10)

Post-WHO-SitRep events are attributed to national health authorities of the destination country: CDC (USA), PHAC (Canada), KCDC (South Korea), Taiwan CDC, China CDC, NZ MoH, Thailand MoPH, Japan MHLW, Cambodia MoH, Vietnam MoH, India NCDC, HK CHP, UAE MoHAP, Mexico SSA, Cook Islands MoH, Vanuatu MoH.

---

## Global Aggregations

- **Global weekly deaths**: Sum of all country weekly deaths.
- **Global weekly cases**: Sum of all country weekly cases. Recalculated after each data modification.
- Countries with `null` case data for a given week are excluded from the sum (not treated as zero).

---

## Summary Table

| Country/Group | Deaths noise | Cases noise | Notes |
|---|---|---|---|
| Vietnam | 30% binomial | — | Severe underreporting |
| Korea, Japan, Thailand | ±10% + null wk 202502 | Full reporting + null wk 202502 | Reporting delays |
| France, UK | EpiHiper (raw) | EpiHiper (raw) | ABM substitution |
| California, Virginia | EpiHiper (raw) | EpiHiper (raw) | VA: deaths zeroed |
| Australia | ±10% | Raw (no noise) | Origin country |
| United States | ±10% | Raw + deaths added | State-level |
| New Zealand | ±10% | Raw (no noise) | Deaths-as-cases rule |
| Singapore | ±10% | 90% binomial | Secondary export hub |
| Fiji | ±10% | 70% binomial | Secondary export hub |
| Italy, Switzerland, Spain, Germany, Ireland, Israel, Canada | ±10% | 90% binomial | |
| Argentina, South Africa, Brazil | ±10% | 50% binomial | Near-zero in window |
| All other GLEAM countries | ±10% | — | No case data |

---

## Timeline Additions (not data-driven)

Two narrative timeline entries added to provide context for team decision-making:

- **January 10**: Several countries discussing travel restrictions to/from China, South Korea, Japan, Singapore, and UK.
- **January 15**: Australian movements advocating for lifting restrictions citing epidemic slowdown; public health officials urge caution noting possible testing saturation.

---

## CSV Downloads

All CSV download options include both deaths and cases data:

- **Global**: week, week_end_date, global_weekly_deaths, global_cumulative_deaths, global_weekly_cases, global_cumulative_cases
- **Single country**: week, week_end_date, weekly_deaths, cumulative_deaths, weekly_cases, cumulative_cases
- **Single US state**: week, week_end_date, weekly_deaths, cumulative_deaths, weekly_cases, cumulative_cases
- **All US states**: week, week_end_date, state, weekly_deaths, weekly_cases

---

*Random seeds: 42 (NumPy) for all stochastic filters on cases, deaths, and initial importation sampling. Seed 123 for the secondary importation sampling run (Far East 10%, other 5%).*
