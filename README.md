# TUMxBLK Seminar
## Heuristic and Modern Portfolio Construction Approaches in Different Volatility Regimes

In this seminar we analysed different portfolio construction approaches, ranging from simple heuristic approaches like a 1/n asset allocation to modern covariance based approaces like the risk parity approach.

Below is a high level description for the logic of each portfolio construction logic.

# 1/n Portfolio Construction with Rebalancing and Transaction Costs

Implementation for constructing a 1/n portfolio using individual asset compounding. The code is flexible as it can simulate a continuously rebalanced portfolio (with optional transaction costs) or a true buy‐and‐hold portfolio (where the initial weights are set only at inception).

## Overview

The code is designed to work with the `returns` dataframe, which contains daily returns (in decimal format) for multiple assets. The portfolio is constructed by:
- **Assigning initial weights:** Each asset is given an initial allocation based on a 1/n strategy.
- **Daily asset compounding:** Each asset’s value is updated daily based on its return.
- **Optional rebalancing:** At specified intervals (e.g., monthly, quarterly, or yearly), the portfolio can be rebalanced back to the target weights. Transaction costs are applied to the trades required for rebalancing.
- **Buy-and-hold mode:** When rebalancing is disabled, the assets compound independently, and the portfolio weights are allowed to drift over time.

## Parameters and Data Assumptions

- **`rebalance_on`**:  
  - `True`: The portfolio is rebalanced at fixed intervals.  
  - `False`: The portfolio follows a true buy‐and‐hold strategy (no rebalancing).

- **`n_freq`**:  
  The rebalancing frequency. Options include:  
  - `'ME'` for monthly end  
  - `'QE'` for quarterly end  
  - `'YE'` for yearly end

- **`tc_rate`**:  
  Transaction cost rate. Set to `0` for a pure buy‐and‐hold scenario, or to a higher value to simulate actual transaction cost.

- **`initial_inv`**:  
  The initial portfolio investment (e.g., \$1,000,000).

- **`weights`**:  
  A dictionary specifying the initial allocation to each asset. In our 8-asset portfolio, the weights sum up to 1.  
  
- **`returns` DataFrame**:  
  This DataFrame has a DateTimeIndex with daily frequency. Each column represents the daily return (in decimals) for one asset.

## Detailed Explanation

### 1. Initialization

- **Rebalancing Dates:**  
  - If `rebalance_on` is `True`, the code resamples the `returns` DataFrame at the specified frequency (`n_freq`) to get the last trading day of each period. These dates are then aligned to actual trading days.  
  - If `rebalance_on` is `False`, the simulation runs over the entire period (using only the start and end dates).

- **Portfolio and Asset Initialization:**  
  - The overall portfolio value (`port_val`) is initialized with `initial_inv`.
  - Each asset’s value is stored in its own time series (`asset_vals[asset]`), initialized to `initial_inv * weights[asset]`.

### 2. Daily Update (Individual Asset Compounding)

- For each trading day, the code updates the value of every asset by multiplying its previous day’s value by `(1 + daily return)`.
- The overall portfolio value on each day is computed by summing the updated values of all assets.

### 3. Rebalancing with Transaction Costs (if enabled)

At the end of each rebalancing period (except the last one), the following steps occur:
  
- **Target Allocation Calculation:**  
  The desired (target) value for each asset is computed as:
  \[
  \text{Target Value} = \text{Total Portfolio Value} \times \text{Asset Weight}
  \]

- **Trade Calculation:**  
  The absolute difference between the current asset value and its target value gives the dollar amount that must be traded.

- **Turnover and Transaction Costs:**  
  - **Turnover** is calculated as:
    \[
    \text{Turnover} = \frac{\text{Total Dollar Amount Traded}}{\text{Total Portfolio Value}}
    \]
    (Note: Some definitions use half this value to avoid double counting buys and sells.)
  - **Transaction Cost** is computed as:
    \[
    \text{Cost} = \text{tc_rate} \times \text{Total Dollar Amount Traded}
    \]
  The computed cost is subtracted from the portfolio value.

- **Rebalancing Adjustment:**  
  After subtracting transaction costs, each asset’s value is reset to the target allocation based on the adjusted total portfolio value.

### 4. Finalization and Output

- **Forward Filling:**  
  The portfolio time series is forward-filled to ensure continuous data.
  
- **DataFrame Construction:**  
  A DataFrame (`df_portfolio`) is created that contains the overall portfolio value along with the individual asset values.

- **Plotting:**  
  The portfolio value over time is plotted with different colors for the overall portfolio and each asset.

- **Output:**  
  The final portfolio value is printed.