# GridFlow: Modelling GB Electricity Dispatch, Prices and Storage

GridFlow is a half-hourly GB electricity-market modelling project developed using 2025 NESO data. It models least-cost generation dispatch, estimates a marginal price proxy and evaluates the effect of higher renewable availability and battery storage can affect operating cost, gas generation, curtailment and system prices.



## Project Overview

The project investigated how increased renewable generation and battery storage can affect the operation of the GB electricity system.
The 2025 NESO dataset contained 17520 half-hourly periods. Using them, the model:
- Constructed a unified demand and generation dataset
- Carried out a least-cost economic dispatch using linear programming
- Derived a marginal price proxy from the power balance constraint
- Modelled a 2GW batter with state of charge constraints
- Compared a 2025 baseline with higher renewable and higher renewable and storage scenarios

The main question under investigation was:

**How do higher renewable availability and battery storage affect generation dispatch, operating costs, gas dependence, curtailment of renewables, and marginal prices in a simplified GB power market model?**



## Data


The project used the 2025 electricity system data (publicly available) from NESO. The main datasets used were:
- **Historic Demand Data:** Which included system demand, transmission system demand, embedded wind, solar generation and other demand variables.
- **Generation Mix Data:** Generation by technology including wind, solar, nuclear, biomass, gas, hydro, coal, storage and imports

The dataset contained 17520 30min settlement periods that covered the entire 2025 year.


### Data Processing

The raw datasets are stored separately from the processed data.

The workflow for processing the datasets included:
- Converting settlement periods into a continuous 30min modelling timestamp
- Basic EDA: Checking for duplicates, missing data and chronological order
- Reconciling demand and generation renewables
- Merging demand and generation datasets into a single unified modelling dataset
- Constructing modelling variables such as underlying demand, total wind generation and residual load.

Underlying demand was defined as:

[
\text{Dispatch Demand}
=
\text{Underlying Demand}
-
\text{Hydro}
-
\text{Coal}
-
\text{Imports}
-
\text{Other}
\]



Hydro, coal, imports and other generation were treated as fixed observed supply whilst wind, solar, nuclear, gas and biomass were optimised.

Processed datasets are stored in: `data/processed/` and the orignal (raw) datasets are stored in `data/raw/`.


## Methodology

### Economic Dispatch

The core model is a least cost economic dispatch problem solved using SciPy's linear programming. For each 30min period, generation from the optimised technologies is selected to minimise the variable operating cost. 


\[
\min \sum_g c_g P_{g,t}\Delta t
\]


where:

- \(P_{g,t}\) is generation from technology \(g\) in period \(t\);
- \(c_g\) is the assumed short-run marginal cost in £/MWh;
- \(\Delta t = 0.5\) hours.


The power balance constraint is:


\[
\sum_g P_{g,t} = D_t
\]

Generation is bounded by the technology specific availability or capacity limits.

The optimised electricity generation technologies are:
- Wind
- Solar
- Nuclear
- Gas
- Biomass


The others were treated as fixed observed supply and removed from dispatch demand before optimisation.

### Technology Assumptions

Wind and solar use was observed as time varying availability proxies. Nuclear, gas and biomass used fixed capacity limits based on observed maximum output in the dataset.


The marginal costs are tabulated below:


| Technology | Marginal Cost (£/MWh) |
|---|---:|
| Solar | 0 |
| Wind | 4 |
| Nuclear | 17 |
| Gas | 80 |
| Biomass | 85 |


It is important to note, these costs are intended as simplified dispatch assumptions instead of deatiled forecasts of plant level operating costs.



### Marginal Price Proxy

A marginal price proxy is derived from the price of the power balance constraint. It represents the estimated increase in minimum system operating costs required to serve an additional unit of demand. This is used as a simplified approximation of wholesale electricity prices instead of being a reproduction of observed GB market prices.


### Battery Storage

Battery storage was modlled as a multi-periods optimsation problems because the battery's state of charge (SOC) can link multiple time periods together.

The battery assumptions are:

- power capacity: **2 GW**;
- energy capacity: **4 GWh**;
- charging efficiency: **95%**;
- discharging efficiency: **95%**;
- initial state of charge: **2 GWh**;
- final state of charge: **2 GWh**.

Battery state of charge evolves according to:

\[
SOC_t =
SOC_{t-1}
+
\eta_c P^{charge}_t\Delta t
-
\frac{P^{discharge}_t\Delta t}{\eta_d}
\]

The power-balance equation becomes:

\[
Generation_t + Discharge_t
=
DispatchDemand_t + Charge_t
\]



The final SOC constraint prevents the model from reducing system cost by emptying the battery at the end of the year.


### Optimisation Solver

The optimisaton problems were solved using `scipy.optimize.linprog` with the HIGHS Linear Programming solver





## Scenarios

Three scenarios were investigated to evaluate the effects of higher renewable availability and batter storage.



| Scenario | Renewable Availability | Battery Storage |
|---|---:|---:|
| **A — Baseline** | 1.0× 2025 wind and solar profiles | None |
| **B — Higher Renewables** | 1.5× wind and solar availability | None |
| **C — Higher Renewables + Storage** | 1.5× wind and solar availability | 2 GW / 4 GWh |



### Scenario A - Baseline

The baseline uses the original 2025 wind and solar availability and no modelled battery storage. It acts as the reference case for comparing system costs, dispatch and marginal prices.



### Scenario B - Higher Renewables
Wind and Solar availability was increased by 50% relative to the 2025 amounts.

\[
Wind^{max}_{t}=1.5\times Wind^{baseline}_{t}
\]

\[
Solar^{max}_{t}=1.5\times Solar^{baseline}_{t}
\]


Electricity demand and conventional generation methods remained the same.

It is important to note, this is a sensitivity scenario instead of a forecast for future GB renewable generation.


### Scenario C - Higher Renewables and Storage

Scenario C uses the same 1.5x wind and solar availability as Scenario B but also includes the 2GW/4GWh battery storage model.

This allowed the analysis to test whether storage can;
- Absorb the would be curtailed renewable generation
- Reduce reliance on gas and biomass
- Lower system operating cost
- Alter marginal pricing behaviour



## Key Results

The scenario analysis demonstrated how higher renewable availability and battery storage can affect modelled system operation

| Metric | A — Baseline | B — 1.5× Renewables | C — 1.5× Renewables + Battery |
|---|---:|---:|---:|
| Operating Cost (£bn) | 6.209 | 3.790 | 3.727 |
| Gas Generation (TWh) | 63.854 | 34.976 | 34.093 |
| Biomass Generation (TWh) | 0.012 | 0.002 | 0.000 |
| Renewable Curtailment (TWh) | 0.0001 | 0.899 | 0.830 |
| Average Marginal Price Proxy (£/MWh) | 71.13 | 47.47 | 47.80 |
| Battery Discharge (TWh) | — | — | 1.099 |


### Main Findings

Increasing wind and solar by 50% produced significant changes relative to the baseline results. 

- Modelled operating costs reduced by nearly **40%** from £6.21 billion to £3.79 billion
- Gas generation was reduced by approximately **45%** from nearly 64 TWh to nearly 35 TWh
- Average marginal price proxy was reduced by nearly **33%** from nearly £71/MWh to £47.50/MWh
- However, nearly 0.90 TWh of renewable availability was curtailed.

Implementing the 2GW/4GWh battery produced further improvements;
- Operating costs fell further
- Gas generation fell by another **0.88 TWh**
- Renewable curtailment from by nearly 8% from 0.899 TWh to 0.830 TWh
- Modelled biomass generation was eliminated entirely
- Battery Discharged nearly **1.10 TWh** over the year

The average marginal price proxy did, however, increase from £47.47/MWh to £47.80/MWh after storage was introduced. This reflects the battery's opportunity value, with storage included the shadow price of the balanced constrain is not always equal to the marginal cost of the generator operating at margin.

Overall, the model suggests that increased renewable availability can significantly reduce dependence on fossil fuel sources and reduce variable operating cost whilst creating periods of renewable surplus. Battery storage helped absorbed some of the surplus and further reduce gas use and system cost.


## Visulisations

### Annual Operating Cost

![Annual Operating Cost by Scenario](results/Annual Operating Cost by Scenario.png)

Higher renewable availability produced the largest reduction in modelling operating cost and battery storage provides an even further reudction

### Gas Generation

![Gas Generation by Scenario](results/gas_generation_by_scenario.png)


Gas generation fell significantly as renewable availability increased. Adding storage reduced gas generation furhter

### Renewable Curtailment

![Renewable Curtailment by Scenario](results/total_curtailment_by_scenario.png)

Curtailment is negligeable in the baseline but does increase as renewable availability is increased. The implementation of battery storage reduces curtailment slightly but does not eliminate it


### Average Marginal Price Proxy

![Average Marginal Price by Scenario](results/average_marginal_price_by_scenario.png)

