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
