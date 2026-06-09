# Dynamic Risk Portfolio Rebalancer (DRPR) – Honest Technical Framework

## Objective

Estimate a loan's **current risk** relative to its **origination risk** and identify loans whose risk profile has materially changed after underwriting.

---

# Layer 1 — Underwriting Anchor

## Purpose

Create a fixed baseline probability of default (Orig_PD) at loan origination.

## Inputs

- grade
- sub_grade
- fico_range_low
- fico_range_high
- annual_inc
- int_rate
- emp_length

## Models

### Primary

Grade/Subgrade PD Lookup

Example:

| Grade | Origination PD |
|---------|---------|
| A | 6.04% |
| B | 13.39% |
| C | 22.44% |
| D | 30.38% |
| E | 38.48% |
| F | 45.20% |
| G | 49.93% |

### Secondary Validation

Logistic Regression

Used to verify and refine lookup-based PD estimates.

## Output

Orig_PD

---

# Layer 2 — Repayment Progression Features

## Purpose

Extract borrower repayment behaviour from LendingClub loan snapshots.

## Features

### Repayment Ratio

repayment_ratio =
total_rec_prncp / loan_amnt

Measures principal recovered.

---

### Interest Recovery Ratio

interest_recovery =
total_rec_int / total_pymnt

Measures proportion of payments going toward interest.

---

### Payment Efficiency

payment_efficiency =
total_rec_prncp / total_pymnt

Measures principal reduction efficiency.

---

### Months Since Last Payment

months_since_last_payment

Proxy for payment recency.

---

### Months Active

months_active

Loan age.

---

### Repayment Deviation

repayment_deviation =
expected_principal(t) -
actual_principal_recovered

Measures distance from expected amortization path.

## Output

Repayment Feature Vector

---

# Layer 3 — Vintage Risk Adjustment

## Purpose

Adjust risk estimates for cohort-level macro effects.

## Inputs

- issue_d
- cohort default rates

## Computation

### Vintage PD

vintage_PD =
charged_off_loans /
total_loans

### Vintage Adjustment Factor

vintage_adj =
vintage_PD /
expected_grade_mix_PD

## Interpretation

| Vintage Adj | Meaning |
|---------|---------|
| >1 | Cohort underperformed |
| <1 | Cohort outperformed |

## Output

Vintage Adjustment Factor

---

# Layer 4 — Latent Risk State Segmentation

## IMPORTANT TECHNICAL NOTE

The LendingClub dataset contains:

- one row per loan
- one cumulative snapshot

There are no monthly observations.

Therefore:

- no true borrower trajectory exists
- no genuine temporal sequence exists
- no meaningful state transitions can be learned

---

## What Was Originally Intended

Hidden Markov Model:

Healthy → Watchlist → Stress → Critical → Default

Learning:

P(State_t+1 | State_t)

from borrower evolution through time.

This requires monthly servicing data.

---

## What The Prototype Actually Does

Each loan is treated as:

```python
sequence_length = 1
```

Example:

```text
Loan A -> [obs]
Loan B -> [obs]
Loan C -> [obs]
```

No transitions exist.

---

## Mathematical Consequence

The HMM collapses into a Gaussian Mixture Model–like clustering system.

The model learns:

- cluster means
- cluster variances
- cluster membership probabilities

It does NOT learn:

- borrower migration
- transition probabilities
- temporal dynamics

---

## Practical Interpretation

Loans are assigned to latent risk clusters:

| Cluster Label | Interpretation |
|---------|---------|
| Healthy | Strong repayment profile |
| Watchlist | Slight deterioration |
| Stress | Moderate deterioration |
| Critical | Severe deterioration |
| Default-like | Extremely weak repayment profile |

These are point-in-time clusters rather than temporal states.

## Output

Latent State Probabilities

Example:

```text
P(Healthy)
P(Watchlist)
P(Stress)
P(Critical)
P(Default)
```

---

# Layer 5 — Dynamic PD Engine

## Purpose

Generate current probability of default.

## Model

LightGBM / XGBoost

## Inputs

Orig_PD

Repayment Features

Latent State Probabilities

Vintage Adjustment Factor

Loan Age

Grade

FICO

## Output

Current_PD

---

# Layer 6 — Risk Drift Engine

## Purpose

Measure change in borrower risk.

Formula:

Risk_Drift =
Current_PD - Orig_PD

## Interpretation

| Drift | Meaning |
|---------|---------|
| < 0 | Improved |
| 0–10% | Watchlist |
| 10–25% | Stress |
| >25% | Critical |

## Output

Risk Drift Score

---

# Layer 7 — Explainability

## Model

SHAP

## Purpose

Explain why Dynamic PD changed.

Example:

```text
+8% repayment_ratio
+5% months_since_last_payment
+3% vintage adjustment
```

## Output

Feature-level risk attribution

---

# Layer 8 — Intervention Engine

## Purpose

Convert risk outputs into actions.

| State | Action |
|---------|---------|
| Healthy | No Action |
| Watchlist | Reminder |
| Stress | Relationship Manager Outreach |
| Critical | Collections Escalation |

## Inputs

- Dynamic PD
- Risk Drift
- Latent State

## Output

Operational Recommendation

---

# Major Limitation

The original architecture describes:

"Borrower migration through hidden states over time."

The LendingClub dataset does not contain the longitudinal data required to support that claim.

Therefore the current prototype supports:

✅ Origination risk estimation

✅ Dynamic PD prediction

✅ Risk drift monitoring

✅ Latent risk segmentation

❌ Genuine HMM state transitions

❌ Borrower migration modelling

❌ Temporal sequence learning

The latent-state layer should therefore be described as:

"Latent Risk State Segmentation"

rather than

"Temporal Hidden Markov Borrower Migration Model."
