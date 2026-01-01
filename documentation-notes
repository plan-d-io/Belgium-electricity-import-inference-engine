# Marginal Technology Inference for Multiple Countries: Methodology and Documentation

## Overview

This document describes the multi-country extension of the marginal technology inference methodology. The tool processes four countries (France, United Kingdom, Netherlands, Germany) with country-specific logic tailored to each market's structure and dispatch behavior.

## Motivation

### Marginal vs Average Technology

In electricity markets, the "marginal technology" is the generation source that sets the market-clearing price at a given moment. Traditional approaches often rely on merit-order assumptions based on fuel costs. However, this can be misleading, especially in markets with strategic dispatch behavior or different generation mixes.

**"Marginal tech" here is a proxy inferred from observable aggregates, not a reconstruction of Euphemia bids, constraints, or unit commitment.**

### Multi-Country Approach

Different countries have distinct market structures and dispatch patterns:

- **France**: Strategic reservoir hydro dispatch based on opportunity cost; large nuclear baseload; storage arbitrage in scarcity
- **United Kingdom & Netherlands**: Gas-dominated marginality; simpler merit-order proxy; less strategic hydro
- **Germany**: Multiple marginal regimes (lignite/coal/gas) with strong RES swings; dynamic switching between coal and gas

The methodology uses country-specific eligibility rules and price-regime imputation fallbacks to account for these differences.

### Two-Stage Methodology: Detection + Price-Regime Imputation

The methodology uses a **two-stage approach** to ensure every interval receives a marginal technology label:

1. **Detection Stage**: Identifies marginal technology from empirical output ramps. This can produce "None/Flat" (no significant ramping) or "Mixed/Unclear" (ramping from non-dispatchable sources).

2. **Price-Regime Imputation Fallback**: When detection fails, imputes marginal technology based on day-ahead price regime and online generation capacity. This leverages the fact that price is the market's marginal value signal.

The price-regime imputation is **transparently labeled** as "price_imputed" (vs "detected") and must be interpreted as an imputation rule, not a unit-level identification.

## Common Building Blocks

### Data Requirements

Each country file must contain:
- `timestamp`: Quarter-hourly timestamps
- `Day-ahead Price (EUR/MWh)`: Price column (€/MWh)
- Generation technology columns (MW): Any numeric columns except timestamp and price are treated as generation technologies

**Robust to missing columns**: If a technology column is missing in a country, it is treated as unavailable; the script does not error.

### Core Computations

1. **First Differences**: `dG_tech[t] = G_tech[t] - G_tech[t-1]`
2. **Positive Ramps**: `pos_dG = max(dG, 0)`
3. **Ramp Noise Threshold**: `delta_threshold_mw = max(50, 0.005 * median(total_generation_mw))`
4. **Online Level Threshold**: `online_threshold_mw = max(200, 0.002 * median(total_generation_mw))`
5. **Price Quantiles**: P30, P70, P90 computed from entire year's price series

### Technology Classifications

**Non-Dispatchable / Exogenous** (excluded from candidates):
- Wind Onshore, Wind Offshore, Solar
- Hydro Run-of-river and pondage, Marine
- Other renewable, Geothermal

**Typically Inframarginal**:
- Nuclear (country-specific gating)

**Dispatchable Candidates**:
- Hydro Water Reservoir, Hydro Pumped Storage
- Fossil Gas, Fossil Hard coal, Fossil Brown coal/Lignite
- Fossil Coal-derived gas, Fossil Oil, Fossil Oil shale, Fossil Peat
- Biomass, Waste, Energy storage, Other

## Country-Specific Logic

### France (FR)

**Market Characteristics:**
- Strategic reservoir hydro dispatch based on opportunity cost
- Large nuclear baseload (typically inframarginal)
- Storage arbitrage in scarcity periods

**Detection Logic:**
- **Storage eligibility**: Only in high-price (P >= P70) or scarcity (P >= P90) regimes
- **Hydro preference**: In high-price hours, prefer Hydro Water Reservoir if its ramp is within 10% of top dispatchable ramp
- **Nuclear gating**: Strict - only if:
  - Nuclear is largest ramp among ALL techs
  - No eligible dispatchable exceeds 50% of nuclear ramp (2.0× rule)
  - Not in scarcity period
- **Storage in scarcity**: Prefer storage if it has largest ramp

**Price-Regime Imputation Fallback:**
- **Scarcity (P >= P90)**: Storage (if online) → Gas → Reservoir → Coal → Nuclear
- **High (P70-P90)**: Reservoir → Gas → Coal → Nuclear
- **Low (P <= P30)**: Nuclear (imputed inframarginal baseline)
- **Mid (P30-P70)**: Gas → Reservoir → Coal → Nuclear

**Rationale**: Reservoir hydro is strategically dispatched in high-price hours; nuclear is almost never marginal.

### United Kingdom (GB)

**Market Characteristics:**
- Gas-dominated marginality
- Little strategic reservoir hydro
- Simpler merit-order proxy works better

**Detection Logic:**
- **Storage eligibility**: Only in high/scarcity price regimes
- **No hydro preference**: Treat Hydro Water Reservoir as normal dispatchable (rare)
- **Nuclear gating**: Very strict (even stricter than FR) - only if no eligible dispatchable exceeds 30% of nuclear ramp
- **Gas primary**: Fossil Gas is primary marginal candidate

**Price-Regime Imputation Fallback:**
- **Scarcity (P >= P90)**: Gas → Storage → Oil → Coal → Other
- **High (P70-P90)**: Gas → Storage → Coal → Other
- **Low (P <= P30)**: Gas → Nuclear (if present) → Other
- **Mid (P30-P70)**: Gas → Other

**Rationale**: UK price formation dominated by gas/peakers rather than reservoir hydro.

### Netherlands (NL)

**Market Characteristics:**
- Gas-dominated, similar to UK
- CHP/"Other" may exist but keep simple
- Marginality is stable and usually gas

**Detection Logic:**
- **Storage eligibility**: Only in high/scarcity price regimes
- **No hydro preference**: No special hydro handling
- **Nuclear gating**: Very strict (if nuclear present)
- **Gas primary**: Fossil Gas is primary marginal candidate

**Price-Regime Imputation Fallback:**
- **Scarcity (P >= P90)**: Gas → Storage → Oil → Coal → Other
- **High (P70-P90)**: Gas → Coal → Other
- **Low (P <= P30)**: Gas → Other (do not force nuclear if NL doesn't have it)
- **Mid (P30-P70)**: Gas → Other

**Rationale**: Marginality is stable and usually gas; simpler than Germany/France.

### Germany (DE)

**Market Characteristics:**
- Multiple marginal regimes (lignite/coal/gas)
- Strong RES swings
- Dynamic switching between coal and gas
- Nuclear absent

**Detection Logic:**
- **Storage eligibility**: Only in high/scarcity price regimes
- **No hydro preference**: No France-style reservoir preference
- **Nuclear gating**: None (nuclear not present)
- **Coal/lignite priority**: Lignite and hard coal can be marginal in mid/low price regimes

**Price-Regime Imputation Fallback:**
- **Scarcity (P >= P90)**: Gas → Storage → Oil → Hard coal → Lignite → Other
- **High (P70-P90)**: Gas → Hard coal → Lignite → Other
- **Mid (P30-P70)**: Hard coal → Lignite → Gas → Other
- **Low (P <= P30)**: Lignite → Hard coal → Other

**Rationale**: Germany often has coal/lignite on the margin in mid/low price regimes; gas tends to dominate high/scarcity.

## Cross-Border Export Attribution to Belgium

### Overview

The export attribution module extends the marginal technology inference to construct **time-varying inferred marginal stacks** for Belgium's neighbouring countries (FR, DE, NL, GB) and allocates export flows to Belgium accordingly. This is **analytical attribution**, not detailed market replay.

### Conceptual Framing

**This script does NOT reconstruct the Euphemia market coupling dispatch.**

Specifically, it does NOT observe:
- Bid/offer curves
- Unit-level dispatch
- Start-up costs
- Internal congestion
- Flow-based domain constraints
- Cross-zonal shadow prices
- Redispatch

Therefore, it cannot recreate the true merit order or price-setting unit.

Instead, the script constructs a **time-varying inferred marginal stack**, intended for **crude but transparent attribution analysis**, based on:

1. **Observed marginal response**: Which technologies visibly increase output at the margin (ΔG > 0)
2. **Price-regime logic**: Which technologies are plausibly price-setting given the day-ahead price level and availability

This inferred stack behaves *like* a merit order for attribution purposes, but should always be considered as *a proxy marginal stack inferred from aggregate observables*.


### Why a Stack is Needed

Export volumes to Belgium often exceed the incremental ramp of a single marginal technology. For example:
- A country may export 2000 MW to Belgium
- The top marginal technology (e.g., Fossil Gas) may only ramp 500 MW
- The remaining 1500 MW must be allocated across additional technologies in the inferred stack

The stack represents the **ordered set of technologies** that could plausibly be exporting, ranked by their marginal response and price-regime logic.

### Method Overview

For each exporting country and each quarter-hour:

#### A) Detected Marginal Response (Top of Stack)

- Compute positive generation ramps: `pos_dG_tech = max(G[t] − G[t−1], 0)`
- Apply country-specific eligibility rules (same as marginal tech inference):
  - Storage gating (only in high/scarcity price)
  - Nuclear hard-gating
  - France reservoir hydro preference
  - Germany coal/lignite regimes
- Rank eligible technologies by `pos_dG` (descending)
- These form the **top of the inferred marginal stack**, with `available_incremental_mw = pos_dG_tech`

#### B) Price-Imputed Extension (Lower Part of Stack)

If export volume exceeds the detected ramps:
- Extend the stack using **price-regime-based ordering** (P30/P70/P90 logic), already defined per country
- Only include technologies that are "online": `generation_level > online_threshold_mw`
- For these technologies, define a **conservative availability proxy**: `available_proxy_mw = min(generation_level, remaining_export_mw)`
- This is NOT incremental capability, but an attribution upper bound

#### C) Allocation Logic

- Walk down the inferred stack: `allocate = min(available_mw, remaining_export_mw)`
- Continue until export volume is covered or candidates exhausted
- Any unallocated remainder is stored explicitly as `residual_mw`

### Distinction: Detected vs Price-Imputed

- **Detected marginal response**: Technologies with observed positive ramps above threshold. These represent **actual incremental output changes** that could be serving exports.
- **Price-imputed marginal ordering**: Technologies ordered by price-regime logic when detection is insufficient. These represent **plausible marginal ordering** based on price levels and online capacity, but not observed ramps.

The method field (`detected` vs `price_imputed`) distinguishes these two sources in the output.

### What This Method Supports

- **Coarse attribution analysis**: Understanding which technologies are likely contributing to exports
- **Narrative construction**: Building stories about cross-border flows (e.g., "French nuclear exports in low-price hours")
- **Aggregate patterns**: Identifying seasonal or price-regime patterns in export attribution
- **Transparency**: Clear documentation of assumptions and limitations

### What This Method Does NOT Support

- **Precise price-setting identification**: Cannot identify the exact unit or bid that sets the export price
- **Welfare analysis**: Cannot compute consumer/producer surplus or market efficiency
- **Congestion analysis**: Cannot identify internal transmission constraints affecting exports
- **Unit-level dispatch**: Cannot determine which specific power plants are exporting
- **Real-time accuracy**: Uses day-ahead data; real-time dispatch may differ

### Export Attribution Outputs

For each neighbour exporting to Belgium:

- `<CC>_exports_to_BE_allocated_2025.parquet` / `.csv`

Each file contains:
- `timestamp`: Quarter-hourly timestamp
- `export_mw`: Total export volume to Belgium (MW)
- `tech_1`, `mw_1`, `method_1`: Top technology, allocated MW, method (detected/price_imputed)
- `tech_2`, `mw_2`, `method_2`: Second technology
- ... (up to `tech_5`, `mw_5`, `method_5` by default, configurable via `--top-k`)
- `residual_mw`: Unallocated remainder (if any)

### Summary Visualizations

For each neighbour:
- Pie chart: "Attributed Exporting Technologies to Belgium – [Country] 2025"
- Aggregates allocated export MWh per technology over the year
- Saved as: `<CC>_export_to_BE_tech_share_2025.png`

### Running Export Attribution

```bash
python attribute_exports_to_BE.py
```

Options:
- `--input-dir`: Directory containing generation and cross-border flow files
- `--output-dir`: Output directory for results
- `--top-k`: Number of top technologies to track (default: 5)
- `--delta-threshold`: Ramp detection threshold (default: computed per country)

## Outputs

### Per-Country Files

For each country (FR, GB, NL, DE):
1. `<CC>_marginal_tech_2025.parquet`: Parquet file with full results
2. `<CC>_marginal_tech_2025.csv`: CSV file with full results
3. `<CC>_marginal_tech_share_final_2025.png`: Pie chart of final marginal tech shares
4. `<CC>_marginal_tech_share_detected_2025.png`: Pie chart of detected marginal tech shares

**Output columns:**
- `timestamp`: Quarter-hourly timestamp (index)
- `price`: Day-ahead price (EUR/MWh)
- `high_price`, `scarcity_price`: Boolean price regime flags
- `price_quantile_regime`: Price regime label ("low", "mid", "high", "scarcity")
- `marginal_tech_detected`: Result from detection stage (can be "None/Flat" or "Mixed/Unclear")
- `marginal_tech_final`: Final marginal technology (always filled)
- `marginal_method`: "detected" or "price_imputed"
- `marginal_confidence`: Confidence level ("high", "medium", "low", "imputed")
- `top1_tech`, `top1_pos_delta_mw`: Top technology by positive ramp (diagnostic)
- `top2_tech`, `top2_pos_delta_mw`: Second-top technology (diagnostic)
- `gas_online`, `reservoir_online`, `coal_online`, `storage_online`: Online flags

### Combined Output

**`marginal_tech_all_countries_2025.csv`**: Combined CSV with shared timestamp index and one column per country:
- `timestamp`
- `FR_marginal_tech`: Final marginal tech for France
- `GB_marginal_tech`: Final marginal tech for United Kingdom
- `NL_marginal_tech`: Final marginal tech for Netherlands
- `DE_marginal_tech`: Final marginal tech for Germany

Timestamps are outer-joined; missing intervals are handled robustly.

## Limitations and Caveats

1. **No Bid Data**: This method does not use actual market bids or unit-level data. It is a proxy based on observed output changes.

2. **No Congestion Information**: Transmission constraints and zonal pricing are not considered. The method assumes a single market price per country.

3. **Cross-Border Flows**: The base marginal tech inference does not model cross-border flows. However, the export attribution module (`attribute_exports_to_BE.py`) extends the methodology to attribute exports to Belgium using inferred marginal stacks.

4. **Proxy Marginality**: The method infers marginality from output changes, not from actual market-clearing mechanisms. It may misclassify in cases where:
   - Multiple technologies ramp simultaneously
   - Ramping is driven by non-price factors (e.g., forced outages)
   - Cross-border flows affect marginal pricing

5. **15-Minute Resolution**: The method uses quarter-hourly data. Finer resolution (e.g., 5-minute) might reveal more nuanced behavior.

6. **Day-Ahead vs Real-Time**: The method uses day-ahead prices. Real-time marginal pricing may differ due to forecast errors and balancing actions.

7. **Price-Regime Imputation Limitations**:
   - Price-imputed labels are **not** unit-level identifications; they are pragmatic imputation rules
   - The imputation assumes that price reflects marginal value, which may not hold during:
     - Extreme scarcity events
     - Market manipulation
     - Congestion-driven price separation
   - The online threshold is a simple capacity check; it does not account for:
     - Unit commitment constraints
     - Minimum stable generation levels
     - Ramp rate limitations

8. **Storage Arbitrage Interpretation**: Storage is only eligible in high/scarcity price regimes. This prevents over-classification but may miss storage arbitrage in mid-price periods (though this is less common).

9. **Country-Specific Assumptions**: Each country's logic is tailored to its market structure. Results may not generalize to other countries without modification.

## Instructions to Run

### Prerequisites

- Python 3.7+
- Required packages: `pandas`, `numpy`, `matplotlib`, `pyarrow` (for parquet support)

Install dependencies:
```bash
pip install pandas numpy matplotlib pyarrow
```

### Basic Usage

```bash
python infer_marginal_tech_multi_country.py
```

This assumes:
- Input files: `FR_generation_with_prices_2025.parquet`, `GB_generation_with_prices_2025.parquet`, `NL_generation_with_prices_2025.parquet`, `DE_generation_with_prices_2025.parquet` in the script directory
- Output directory: script directory

### Custom Input/Output

```bash
python infer_marginal_tech_multi_country.py \
    --input-dir /path/to/input \
    --output-dir /path/to/output
```

### Custom Delta Threshold

```bash
python infer_marginal_tech_multi_country.py --delta-threshold 100
```

This sets the ramp detection threshold to 100 MW for all countries (instead of the default per-country formula).

### Expected Runtime

For four countries with full year of quarter-hourly data (~35,000 intervals each), the script typically runs in 1-3 minutes on a modern laptop.

### Console Output

The script prints:
- Per-country summaries:
  - Number of intervals processed
  - Computed P30, P70, P90 thresholds
  - Delta and online thresholds
  - Method breakdown (detected vs price-imputed)
  - Top 10 marginal technology shares
- Combined file creation confirmation

### Produced Files

**Per-country (4 countries × 4 files = 16 files):**
- `<CC>_marginal_tech_2025.parquet`
- `<CC>_marginal_tech_2025.csv`
- `<CC>_marginal_tech_share_final_2025.png`
- `<CC>_marginal_tech_share_detected_2025.png`

**Combined:**
- `marginal_tech_all_countries_2025.csv`

**Total: 17 files**

## Belgium Net-Import Mix: Accounting Mix vs Marginal Attribution Proxy

### Overview

The Belgium net-import mix analysis module (`be_net_import_mix.py`) extends the export attribution methodology to compute two complementary metrics for Belgium's net import position. This module uses the inferred marginal export stacks from neighbouring countries (FR, DE, NL, GB) to construct technology-level mixes for Belgium's net imports.

**This analysis is NOT a physical flow tracing or counterfactual market simulation.** It is an accounting and attribution framework that uses inferred marginal stacks to construct indicative technology mixes.

### Input Data

The module requires:

1. **Cross-border flows**: `BE_crossborder_flows_2025.csv`
   - Columns: `timestamp`, `BE -> DE`, `BE -> FR`, `BE -> GB`, `BE -> NL`, `DE -> BE`, `FR -> BE`, `GB -> BE`, `NL -> BE`
   - MW values, quarter-hourly timestamps with timezone

2. **Export allocation files** (4 files):
   - `FR_exports_to_BE_allocated_2025.csv`
   - `DE_exports_to_BE_allocated_2025.csv`
   - `NL_exports_to_BE_allocated_2025.csv`
   - `GB_exports_to_BE_allocated_2025.csv`
   - Each contains: `timestamp`, `export_mw`, `tech_1`, `mw_1`, `method_1`, ..., `tech_5`, `mw_5`, `method_5`, `residual_mw`

### Metric 1: Net-Import Mix Using Inferred Marginal Export Stacks (Accounting View)

**Purpose**: Construct a net-position-weighted mix of inferred exporting marginal stacks. This is an accounting framework, not a physical generation mix.

**Method**:

For each quarter-hour timestamp:

1. **Compute net import position**:
   ```
   total_import_to_BE = (FR->BE + DE->BE + NL->BE + GB->BE)
   total_export_from_BE = (BE->FR + BE->DE + BE->NL + BE->GB)
   net_import_BE = max(total_import_to_BE - total_export_from_BE, 0)
   ```

2. **If net_import_BE == 0**: The mix is empty/zero for that interval.

3. **If net_import_BE > 0**: 
   - Compute weights for each neighbour based on their gross imports to BE:
     ```
     w_FR = (FR->BE) / total_import_to_BE
     w_DE = (DE->BE) / total_import_to_BE
     w_NL = (NL->BE) / total_import_to_BE
     w_GB = (GB->BE) / total_import_to_BE
     ```
     (Only for neighbours with positive imports)

   - For each neighbour, reconstruct the technology→MW vector from the allocation file (summing `mw_i` over all `tech_i` slots, including residual if present).

   - Form a weighted sum of technology vectors:
     ```
     mix_BE_tech_MW = Σ_c w_c * alloc_c_tech_MW
     ```
     where `alloc_c_tech_MW` is the technology vector from country `c`'s allocation file.

   - Scale the resulting vector so that the sum equals `net_import_BE`:
     ```
     mix_scaled = mix * (net_import_BE / total_import_to_BE)
     ```
     This scaling is necessary because the neighbour allocations sum to `total_import_to_BE`, not `net_import_BE`.

**Interpretation**: This metric provides a technology-level breakdown of Belgium's net imports, weighted by the inferred marginal stacks of exporting neighbours. It is an accounting view: it attributes net imports to technologies based on the inferred marginal response of exporting countries, but does not claim physical flow tracing.

### Metric 2: Marginal Import Attribution (Causal Proxy with Transit Correction)

**Purpose**: Approximate which exporting marginal technologies are activated "because Belgium is a sink", acknowledging that Belgium also acts as a transit zone.

**Method**:

1. **Compute transit ratio**:
   ```
   transit_ratio = min(total_import_to_BE, total_export_from_BE) / max(total_import_to_BE, total_export_from_BE)
   ```
   (Define as 0 if both are zero)

   The transit ratio approaches 1.0 when imports ≈ exports (Belgium is primarily transiting), and approaches 0.0 when Belgium is a net sink.

2. **Define causal weight**:
   ```
   causal_weight = 1 - transit_ratio
   ```

3. **Compute marginal attribution mix**:
   ```
   marginal_attrib_mix = mix_scaled (from Metric 1) * causal_weight
   ```

**Interpretation**: This metric downweights hours where Belgium is mostly acting as transit (imports ≈ exports). When `transit_ratio` is high, the causal weight is low, reflecting that exports from neighbours may not be "caused" by Belgium's net import position but rather by transit flows.

**Limitations**: This is a proxy, not a counterfactual Euphemia re-run. It cannot:
- Identify true physical flow paths
- Determine which specific units are price-setting
- Compute welfare effects or counterfactual market outcomes
- Account for complex market coupling dynamics

### Outputs

#### 1. Per-Timestamp Wide Table

**Files**:
- `BE_net_import_mix_2025.parquet`
- `BE_net_import_mix_2025.csv`

**Columns**:
- `timestamp` (index)
- `total_import_to_BE`, `total_export_from_BE`, `net_import_BE`
- `transit_ratio`, `causal_weight`
- `M1_<techname>_MW`: Metric 1 values per technology
- `M2_<techname>_MW`: Metric 2 values per technology

#### 2. Yearly Aggregated Summary

**File**: `BE_net_import_mix_summary_2025.csv`

**Columns**:
- `tech`: Technology name
- `M1_MWh`: Total MWh from Metric 1
- `M2_MWh`: Total MWh from Metric 2
- `M1_share`: Percentage share for Metric 1
- `M2_share`: Percentage share for Metric 2

Sorted by `M1_MWh` descending.

#### 3. Visualizations

**A) Donut Charts** (year total MWh by tech):
- `BE_net_import_mix_M1_share_2025.png`: Metric 1 share by tech
- `BE_net_import_mix_M2_share_2025.png`: Metric 2 share by tech
- Shows top N techs (default 8) and groups the rest as "Other"
- Legend outside plot

**B) Stacked Area Chart** (weekly aggregation):
- `BE_net_import_mix_M1_stacked_weekly_2025.png`
- x-axis: time (weekly)
- y-axis: net import MWh per week
- stack by tech (top techs + Other)

**C) Transit Diagnostics**:
- `BE_net_import_transit_diagnostics_2025.png`
- x-axis: `net_import_BE`
- y-axis: `transit_ratio`
- Hexbin density plot colored by interval count
- Shows relationship between net import and transit behavior

### What This Method Can Support

- **Coarse narratives**: Understanding which technologies are likely contributing to Belgium's net imports
- **Indicative mix patterns**: Identifying seasonal or price-regime patterns in net import technology composition
- **Transparency**: Clear documentation of assumptions and limitations
- **Accounting framework**: Net-position-weighted attribution using inferred marginal stacks

### What This Method Cannot Support

- **True physical flow tracing**: Cannot determine actual physical paths of electricity flows
- **True price-setting unit identification**: Cannot identify which specific units set prices
- **Welfare/counterfactual claims**: Cannot compute consumer/producer surplus or market efficiency
- **Detailed market replay**: Does not reconstruct Euphemia dispatch or unit-level decisions
- **Congestion analysis**: Cannot identify internal transmission constraints affecting flows

### Running the Analysis

```bash
python be_net_import_mix.py
```

**Options**:
- `--input-dir`: Directory containing flows and allocation files (default: script directory)
- `--output-dir`: Output directory for results (default: script directory)
- `--top-k`: Number of tech slots in allocation files (default: 5)
- `--top-n`: Number of top technologies to show in charts (default: 8)

### Technical Notes

- **Timestamp alignment**: Uses inner join on timestamps; prints warnings if any file has missing timestamps relative to flows
- **Technology name preservation**: Technology names remain exactly as in source files (e.g., "Fossil Gas", "Hydro Water Reservoir") to ensure merges across neighbours work correctly
- **Residual handling**: `residual_mw` from allocation files is optionally included as "Residual/Unallocated" technology
- **MW to MWh conversion**: Uses `MWh = MW * 0.25` for quarter-hour intervals
- **Robustness**: Handles missing timestamps, empty allocation slots, and zero net import intervals gracefully

## References and Further Reading

- ENTSO-E Transparency Platform: Generation and price data
- Euphemia algorithm: Day-ahead market clearing mechanism
- Merit-order effect: Relationship between generation mix and prices
- Opportunity cost of hydro: Strategic reservoir dispatch behavior
- Cross-border market coupling: Impact on marginal pricing

---

**Document Version**: 2025  
**Script Version**: 1.0 (Multi-Country)  
**Last Updated**: 2025

