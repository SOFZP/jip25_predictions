# Jito JIP-25 Validator Ranking
This repository contains automated rankings of Solana validators based on a model inspired by Jito's [JIP-25 proposition](https://forum.jito.network/t/jip-25-expand-the-validator-set-and-modify-jito-stake-pool-eligibility-and-ranking-criteria/877). The data is generated and updated regularly by a custom script, which aggregates metrics from multiple on-chain and off-chain sources.

---

## 📊 Web Dashboard

The data from this repository powers an interactive web dashboard where you can easily view and analyze the Jito JIP-25 rankings.

> **Explore the dashboard: [cryptovik.info/jip25-predictions](https://cryptovik.info/jip25-predictions)**

This project serves as a specialized dataset and will be integrated as an extension into the main Solana dashboard in the future: [cryptovik.info/solana-stakepools-dashboard](https://cryptovik.info/solana-stakepools-dashboard).

---

## ► Data Methodology

The ranking is determined by a sequence of eligibility checks and performance metrics. The script first filters out ineligible validators and then ranks the remaining ones based on four key criteria in order of priority.

### 1. Eligibility Criteria (Ineligibility Flags)

To align with the Jito Network's standards, eligibility is determined by analyzing historical `InstantUnstakeComponents` events from the Jito steward API. A validator is marked as **ineligible** and moved to the bottom of the ranking if it has active negative flags.

The logic for checking flags is as follows:
- **`is_blacklisted`**: The script performs a deep historical search (up to 1000 events) to find the **earliest epoch** a validator was blacklisted. This status is considered permanent.
- **Temporary Flags**: For other incidents, their effect is considered temporary. The validator is marked ineligible only if the incident occurred within the **last 30 epochs**:
    - `delinquency_check` (poor uptime or voting performance).
    - `commission_check` (commission was set too high).
    - `mev_commission_check` (MEV commission was set too high).

The final JSON data includes an `incident_details` field, specifying each active incident and the epoch it was first detected (e.g., `HighCommission:829;Blacklisted:833`).

### 2. Ranking Metrics (for Eligible Validators)

Eligible validators are ranked sequentially based on the following metrics. Lower is better for commissions; higher is better for age and credits.

**1. Maximum Commission (`max_commission`)**
- **Description**: The highest commission rate over the last 30 epochs.
- **Primary Source**: Historical `commission_max_observed` from the **Marinade.Finance API**.
- **Fallback Source**: Current commission from the **Solana CLI**.

**2. Maximum MEV Commission (`max_mev_commission`)**
- **Description**: The highest MEV commission over the last 30 epochs.
- **Primary Source**: Historical `mevCommission` from **TheValidators.io History API**.
- **Fallback Source**: Current `mev_commission_bps` from the **Jito Geyser API**.
- *Note: The logic correctly handles cases where a validator has no Jito blocks in an epoch, using the last known commission instead of incorrectly assigning a 100% penalty.*

**3. Validator Age (`validator_age`)**
- **Description**: The validator's age in epochs.
- **Primary Source**: `first_epoch_distance` from the **Stakewiz.com API**.
- **Fallback Source**: Calculated from `start_epoch` via the **Marinade.Finance API**.

**4. Total Credits (`total_credits`)**
- **Description**: The sum of voting credits over the last 30 epochs.
- **Primary Source**: Sum of historical `credits` from the **Marinade.Finance API**.

---

## How to Use This Data

The data is organized by cluster and epoch in the `/data` directory of this repository.

- **Direct Link to Latest Data**: For easy programmatic access, the `manifest.json` file in each cluster's directory always contains a URL pointing to the most recent data file.
  - Example: `data/mainnet-beta/manifest.json`
- **Historical Data**: All previous runs are stored in their respective epoch folders.
  - Example: `data/mainnet-beta/837/mainnet-beta-jito-ranking-....json`
