# Know Your Resilience (KYR) Scorecard

This repository hosts the interactive executive simulator for the Know Your Resilience (KYR) index, developed at Canada's Financial Wellness Lab. The tool provides a real-time, interactive visualization of the V4 Hybrid Machine Learning calibration, translating raw transactional data into a definitive 0-1,000 resilience score.

## The Resilience Tiers (0 - 1000 Scale)
The KYR algorithm normalizes aggregate behavioral indicators into a standardized index, categorizing households into four actionable resilience tiers:
* **Tier 1: Highly Resilient (750 - 1000):** Substantial shock-absorption capacity, high spending flexibility, and strong amortizing debt repayment.
* **Tier 2: Stable (600 - 749):** Adequate cash flows and baseline buffers, but vulnerable to prolonged structural shocks (e.g., house-rich but cash-poor).
* **Tier 3: Vulnerable (450 - 599):** Systemic cash flow exhaustion, high reliance on fixed obligations, and inadequate liquid asset buffers.
* **Tier 4: Distressed (0 - 449):** Active delinquency, massive outflow volatility, and high debt-service burdens.

## Data Mapping: Raw Variables to Behavioral Indicators
The V4 architecture maps highly granular credit union banking data into five theoretical pillars.

### 1. Cash In (Income & Inflows)
* **Adequate Inflows & Trend:** Aggregates `TransactionCategory == 'Depositing'` (standard payroll) with `TransactionType == 'Transfer In'` (e-transfers/gig-income) to capture the true digital income baseline.
* **Income Volatility:** Calculates the 6-month standard deviation of inflows. Highly volatile income (e.g., erratic contract work) heavily penalizes the score.

### 2. Cash Out (Consumption & Flexibility)
* **Inflexible Spending (Fixed):** Captures rigid structural costs using `TransactionCategory == 'Paying Bills'` and `TransactionChannel == 'System'`.
* **Spending Flexibility:** Captures adaptable consumption using `TransactionChannel == 'POS'` and `'ATM'`. A high POS ratio rewards the member for possessing a financial "throttle" they can pull back during a shock.
* **Spending Volatility:** The 6-month standard deviation of outflows. In the V4 model, this is the single highest-weighted cash-flow indicator; extreme consumption spikes reliably predict overdrafts.

### 3. Savings Behaviour
* **Recurring Savings & Persistence:** Evaluates `contribution_month_X` across both internal and external investment data (`inv_in_df` and `inv_out_df`). Rewards consistency (months out of 24) over pure volume.
* **Automated Discipline:** Boolean check on `auto_withdrawal_enabled_current` to reward systemic saving habits.

### 4. Liquidity & Assets
* **Adequate Buffer:** Measures `demand_df` balances (Share/Chequing/Savings) against the member's 24-month average monthly outflow.
* **Inaccessible Liquidity:** Triggers a severe penalty if illiquid assets (`los_df` property values) are more than double the accessible liquid cash.

### 5. Debt & Repayment
* **TDS Ratio & Amortization:** Calculates fixed debt burden using `tds_at_origination` and evaluates `payment_for_principal` to penalize interest-only traps.
* **Repayment Integrity:** Analyzes `DelinquencyStatus` (`max_delinq`). A missed payment is the strongest negative tipping point in the entire scorecard framework.

---

## V4 Hybrid Calibration Weights
The following tables reflect the exact +/- point allocations derived from the constrained logistic regression calibration against the active, addressable Mainstreet cohort (13,128 members).

### Positive Rewards (+412 Max)
| Pillar | Indicator | Empirical Weight | Hybrid Point Allocation |
| :--- | :--- | :--- | :--- |
| **Cash In** | Adequate Inflows | 33.3% | +20 |
| **Cash In** | Low Vol Inflows | 33.3% | +20 |
| **Cash In** | Stable Inflow Trend | 33.3% | +20 |
| **Cash Out** | Stable Spending | 96.1% | +48 |
| **Cash Out** | Spending Flexibility | 1.9% | +11 |
| **Cash Out** | Positive Cash Flow | 1.9% | +11 |
| **Savings** | Recurring Savings | 57.1% | +44 |
| **Savings** | Increasing Savings | 42.9% | +36 |
| **Savings** | Savings Persistence | 0.0% | +10 |
| **Savings** | Auto Transfers | 0.0% | +10 |
| **Liquidity** | Adequate Buffer | 74.2% | +51 |
| **Liquidity** | Accessible Liquidity | 25.8% | +24 |
| **Liquidity** | Positive Net Worth | 0.0% | +10 |
| **Liquidity** | Improving Net Worth | 0.0% | +10 |
| **Debt** | Strong Repayment | 99.7% | +67 |
| **Debt** | Low Debt Service | 0.2% | +10 |
| **Debt** | Amortizing Debt | 0.0% | +10 |

### Negative Penalties (-410 Max)
| Pillar | Indicator | Empirical Weight | Hybrid Point Allocation |
| :--- | :--- | :--- | :--- |
| **Cash In** | Inadequate Inflows | 33.3% | -20 |
| **Cash In** | High Vol Inflows | 33.3% | -20 |
| **Cash In** | Declining Inflows | 33.3% | -20 |
| **Cash Out** | Volatile Spending | 94.5% | -48 |
| **Cash Out** | Inflexible Spending | 2.8% | -11 |
| **Cash Out** | Negative Cash Flow | 2.8% | -11 |
| **Savings** | Periodic Savings | 55.0% | -43 |
| **Savings** | Decreasing Savings | 45.0% | -37 |
| **Savings** | Interrupted Savings | 0.0% | -10 |
| **Savings** | No Auto Transfers | 0.0% | -10 |
| **Liquidity** | Inadequate Buffer | 59.0% | -42 |
| **Liquidity** | Inaccessible Liquidity | 36.3% | -30 |
| **Liquidity** | Decreasing Net Worth | 4.8% | -13 |
| **Liquidity** | Negative/0 Net Worth | 0.0% | -10 |
| **Debt** | Weak Repayment | 96.2% | -63 |
| **Debt** | High Debt Service | 3.8% | -12 |
| **Debt** | Interest Only | 0.0% | -10 |
