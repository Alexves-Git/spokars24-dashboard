# SPOKARS-24 Dashboard — Data Filters and Noise Documentation

**Internal reference only. Do not distribute to exercise teams.**

This document describes all noise filters, reporting adjustments, and data transformations applied to the SPOKARS-24 Surveillance Dashboard. These details are not disclosed to the teams participating in the C-CHA tabletop exercise.

---

## Data Sources

The dashboard integrates data from two simulation frameworks:

- **GLEAM** (Global Epidemic and Mobility Model): 3,253 subpopulations, 232 countries, 36 weeks of simulation. Source file: `master_world-cities.csv` (~787K rows). Used for global death and case data.
- **EpiHiper**: Agent-based model used for four locations (California, Virginia, France, United Kingdom), initialized with GLEAM importations. EpiHiper output replaces GLEAM data entirely for these locations.

### Dashboard Time Window

Epi weeks 202446 through 202502 (November 16, 2024 – January 11, 2025), 9 weeks total.

### Death Field

`Deaths_total_delayed` (not `Deaths_total`) — incorporates realistic reporting delays from the GLEAM simulation.

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

- Source: GLEAM `Infectious_symp_total_detected_delayed` field.
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
| Italy, Switzerland, Spain, Germany, Ireland, Israel, Canada | ±10% | 90% binomial | |
| Argentina, South Africa, Brazil | ±10% | 50% binomial | Near-zero in window |
| All other GLEAM countries | ±10% | — | No case data |

---

*Random seed: 42 (NumPy) for all stochastic filters.*
