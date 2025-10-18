# Monte Carlo Simulation for Global Headlamp Sourcing Risk (Tesla Supply Chain Case)

## Overview

This project builds a **risk-adjusted total landed cost (TLC)** model using Monte Carlo simulation to evaluate sourcing strategies for an automotive headlamp supplier under geopolitical, operational, and logistics uncertainty.

Each supplier region (U.S., Mexico, China) has unique cost drivers — tariffs, yield variability, and logistics volatility — modeled probabilistically to support **data-driven sourcing and resilience decisions**.

---

## Objectives

* Quantify the **expected (P50)** and **risk-adjusted (P90)** unit costs for each sourcing location.
* Integrate realistic uncertainties:

  * Yield risk (Beta-distributed, driven by automation, skill, and maturity indices)
  * Tariff/geopolitical risk (discrete scenario probabilities)
  * Logistics volatility (lognormal, scaled by transit days)
* Recommend the **optimal sourcing location** that minimizes the P90 cost (resilient strategy).

---

## Model Architecture

| Component                            | Description                                                                                                               | Distribution             |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| **Yield (Manufacturing Efficiency)** | Converts operational maturity indices (A/H/M) → expected yield via logistic function; uncertainty captured by Beta(α, β). | **Beta**                 |
| **Tariff & Geopolitical Risk**       | Randomized based on policy and trade scenarios (e.g., USMCA compliance, Section 232).                                     | **Discrete categorical** |
| **Logistics Volatility**             | Cost multiplier tied to transit days via log-linear σ; sampled lognormal for daily variation.                             | **Lognormal**            |

---

## Simulation Parameters

* **10 000 runs per site**
* **Transit days:** US = 5, MX = 9, CN = 32
* **Base Ex-Works Costs:** $71 (US), $55 (MX), $47 (CN)
* **Tariff assumptions:**

  * US = 0 %
  * MX = 8 % chance of 25% tariff (non-USMCA)
  * CN = 95 % at $15 (25% tariff), 5 % at $59(100% tariff) escalation
* **Outputs:**

  * Mean, Median (P50)
  * P90 (risk-adjusted cost)
  * Strategic recommendation (lowest P90 = optimal sourcing)

---

## Results (10 000 Simulations)

| Metric                  | US (USD) | Mexico (USD) | China (USD) |
| :---------------------- | -------: | -----------: | ----------: |
| **Mean**                |    82.04 |    **65.70** |       76.89 |
| **Median (P50)**        |    81.97 |    **64.47** |       74.66 |
| **P90 (Risk-Adjusted)** |    83.57 |    **67.47** |       78.41 |

**Strategic Recommendation:** **Mexico** offers the lowest P90 ($ 67.47), balancing efficiency and resilience.
* **China** has lowest mean cost but heavy-tail risk from tariffs / policy shocks.
* **Mexico** combines competitive cost with stable tail risk via USMCA access and shorter logistics.
* **U.S.** avoids tariffs entirely but faces high domestic labor + energy cost.

---

## Key Insights

* **Resilience premium:** Paying slightly more per unit in Mexico yields lower exposure to tail risks (tariff or logistics shocks).
* **Transit-based volatility:** Longer ocean lanes exhibit heavier right-tailed cost risk, validating the use of lognormal scaling.
* **Dynamic yield modeling:** Factory automation and workforce maturity directly influence yield distribution via logistic–Beta integration.

---

## Appendix: Methodology & Key Equations

### 1. Yield Risk Modeling (Beta Distribution)

Production yield during ramp-up affects unit cost significantly. Instead of assigning ad-hoc probabilities, yield is derived from **site maturity attributes**:

$$
\mu_s = \sigma(\beta_0 + \beta_A A_s + \beta_H H_s + \beta_M M_s), \quad 
\sigma(x) = \frac{1}{1 + e^{-x}}
$$

- \(A_s\): Automation level  
- \(H_s\): Workforce skill / human capital  
- \(M_s\): Manufacturing maturity  
- \(\beta_0, \beta_i\): Calibrated logistic weights (default –1.5, 2.2)

This gives the expected yield \(\mu_s\), which parameterizes the Beta distribution:

$$
\alpha = \mu_s \nu, \qquad 
\beta = (1 - \mu_s)\nu
$$

where \(\nu\) controls variance (higher \(\nu\) = more stable process).

---

### 2. Tariff & Geopolitical Risk (Categorical Scenarios)

Each region draws a random tariff scenario per simulation:

| Site       | Scenario                      | Probability |
| ---------- | ----------------------------- | ----------- |
| **US**     | $0 (domestic)                 | 100 %       |
| **Mexico** | $15.5 (high tariff)           | 8 %         |
|            | $0 (USMCA-compliant)          | 92 %        |
| **China**  | $15 (current)                 | 95 %        |
|            | $59 (geopolitical escalation) | 5 %         |

Probabilities reference **INA (2024)** and **USTR Section 232** trade outlooks.

---

### 3. Logistics Volatility (Lognormal)

Shipping-cost volatility is modeled as a **lognormal multiplier** on base logistics cost:

$$
C_{\text{log}} = C_{\text{base}} \, \exp(\mu + \sigma Z), \qquad Z \sim N(0, 1)
$$

with \(\mu = -0.5\sigma^2\) so that \(E[C] \approx C_{\text{base}}\).  
\(\sigma\) grows with **transit days (T)** by a log-linear rule:

$$
\sigma(T) = a + b \ln(T), \qquad a = 0.0185, \; b = 0.0454
$$

Anchors: short-haul (≈2 days → σ≈0.05) vs. trans-Pacific (≈35 days → σ≈0.18).
These align with volatility seen in **Freightos Baltic Index** and **Drewry WCI** 2023-2025.

---

### 4. Simulation Process

Each Monte Carlo run (10 000 iterations per site):

1. Draw yield ∼ Beta(α, β)
2. Compute Ex-Works / yield → risk-adjusted manufacturing cost
3. Draw logistics multiplier ∼ LogNormal(μ, σ(T))
4. Sample tariff scenario ∼ Categorical(probabilities)
5. Sum to obtain one Total Landed Cost (TLC)

Resulting TLC distributions provide:

* **P50 (median)** – expected cost
* **P90** – risk-adjusted “stress” cost

---

### 5. Citations

* **KPMG Cost of Capital Study 2024** – Automotive WACC ≈ 9.3 %
* **NYU Stern (2024)** – Auto Parts Sector WACC ≈ 10 %
* **Freightos Baltic Index (2023-2025)** – Container-rate volatility > 50 % YoY
* **USTR Section 232 Proclamations (2024-2025)** – Tariff eligibility under USMCA
* **INA (Mexico Auto Parts Industry 2024)** – Export compliance & tariff exemption rates

---
