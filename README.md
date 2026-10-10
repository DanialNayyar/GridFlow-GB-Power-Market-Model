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

















