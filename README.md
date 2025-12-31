# Belgium-electricity-import-inference-engine
A method and tool to infer the make-up of Belgian electricity imports


## What Does Belgium Really Import?


## Motivation
Belgium is structurally interconnected with its neighbours and frequently imports electricity. Headlines often state that *“Belgium imports electricity from France, the Netherlands or Germany”*, but they rarely clarify **what kind of electricity** this actually is. Is Belgium importing:
-	nuclear electricity from France?
-	gas-fired electricity from the Netherlands?
-	coal-fired electricity from Germany?
-	or hydroelectricity dispatched opportunistically?

Answering this question is not trivial. Electricity flows are simultaneous, bidirectional, and shaped by market coupling across Europe. Belgium is not only an importer, but also regularly a transit country, with power flowing through its grid between neighbouring zones.

On social media, many people have strong opinions on the make-up of these import flows, without actually having any clue on how they are *actually* made up. I wanted to get some real numbers, so I started coding. The goal of this little project is not to reconstruct the European market dispatch in detail, but to make a transparent, defensible attempt to answer a simpler but policy-relevant question: *What does the electricity Belgium relies on from abroad most likely consist of?*

## Why the Import Mix Must Be Inferred

Electricity markets differ fundamentally from other commodity trade. Electricity imports:
- cannot be traced to individual power plants,
-	flows may enter Belgium and leave again immediately,
-	generation in neighbouring countries may be driven by demand elsewhere.

Reconstructing true dispatch would require full bid stacks, Euphemia, flow-based constraints, and counterfactual simulations — none of which are publicly available. This means there is no single **“correct”** import mix. Any answer must be an inference, based on assumptions. The objective is therefore **approximation**, not reconstruction. Below I discuss the methodlogical framework/pipeline I created for this.

### Step 1 — Inferring Marginal Technologies per Country
A country does not import the *average* electricity mix of another country. When electricity demand changes, not all power plants respond equally:
- some plants run almost continuously,
- others are adjusted up and down to balance the system.

When demand in one country increases, exports are supplied by the power plants in the exporting country that can still adjust their output at the margin (the so-called *marginal unit*), not by all generators equally. The technologies that respond to short-term changes are the ones that:
- determine prices,
- enable exports,
- and are relevant for questions about imports and dependency.

Identifying these marginal technologies is therefore essential to understand which types of generation are actually activated by imports. This is not trivial.


#### Step 1a — Ramping-Based Detection

At quarter-hourly resolution, marginal behaviour is proxied by changes in output.

For technology i:

ΔGᵢ(t) = Gᵢ(t) − Gᵢ(t−1)

Only positive ramps are considered:

ΔGᵢ⁺(t) = max(ΔGᵢ(t), 0)

Technologies with the largest significant positive ramps are candidates for marginality.

Small fluctuations are filtered using minimum ramp thresholds to avoid noise.

#### Step 1b — Eligibility Rules

Not all ramping technologies are economically marginal:
- wind and solar move exogenously,
- nuclear often ramps but remains inframarginal,
- storage responds strongly but only under certain price conditions.

Eligibility rules are applied per country, reflecting known system characteristics:
- France: strategic reservoir hydro, nuclear baseload
- UK & Netherlands: gas-dominated marginality
- Germany: lignite/coal/gas regime switching

#### Step 1c — Price-Imputed Fallback

In many intervals, ramps are too small or ambiguous. When ramp-based detection fails, marginality is inferred from **price regimes**.

Prices are grouped into low, mid, high, and scarcity regimes. For each regime and country, a plausible marginal ordering is defined based on cost structures and historical dispatch behaviour.

Crucially, this fallback is used **only when ramp-based detection yields no clear result**.

### Step 2 — Constructing an Inferred Marginal Stack
Export volumes frequently exceed the ramp of a single technology. Therefore, a **marginal stack** is constructed:
1. detected marginal technologies (ordered by ramp size),
2. followed by price-imputed technologies ordered by plausibility and availability.

This stack acts as a **proxy merit order**, inferred from observable data rather than bids.

### Step 3 — Allocating Cross-Border Exports

Exports to Belgium are allocated sequentially across the marginal stack until fully attributed. Any residual is recorded explicitly.

### Step 4 — Net Imports and Transit

Belgium’s net import is defined as:

Net Import_BE(t) = max( Σ_c Import_(c→BE)(t) − Σ_c Export_(BE→c)(t), 0 )

Two metrics are reported:
- an accounting-based net import mix,
- a transit-corrected marginal attribution proxy.


## Interpretation

This methodology provides:
- transparency,
- reproducibility,
- and informed approximation.

It does not provide:
- dispatch reconstruction,
- welfare analysis,
- or Euphemia replication.

---

## Data Sources

- ENTSO-E Transparency Platform  
- NESO (UK)  
- Nationaal Energie Dashboard (NL)

---

## License

This document is licensed under the  
**Creative Commons Attribution 4.0 International (CC BY 4.0)**.

© Joannes Laveyne, 2025  
https://creativecommons.org/licenses/by/4.0/
