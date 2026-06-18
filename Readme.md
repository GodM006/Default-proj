https://drive.google.com/file/d/1ZtNn_WdFJxZBzgr6apXxoELX-YlfvVCm/view?usp=sharing

# Dynamic Risk Portfolio Rebalancer
## Enterprise Architecture Blueprint

---

# Executive Summary

The Dynamic Risk Portfolio Rebalancer is a real-time credit risk migration and intervention platform designed to continuously monitor borrower health after loan origination.

Unlike traditional underwriting systems that assign a static risk grade at loan approval, this platform tracks behavioral deterioration, updates the Probability of Default (PD) dynamically, and triggers preventive interventions before formal delinquency or charge-off occurs.

---

# Core Business Problem

Current banking workflow:

```text
Application
     ↓
Underwriting
     ↓
Loan Approval
     ↓
(No Monitoring)
     ↓
Default
     ↓
Collections
```

Problems:

- Static underwriting assumptions become stale.
- Borrowers deteriorate unnoticed.
- Collections react after damage occurs.
- High provisioning and recovery costs.

---

# Proposed Workflow

```text
Application
     ↓
Underwriting
     ↓
Baseline Risk Anchor
     ↓
Behavior Monitoring
     ↓
Dynamic PD Recalculation
     ↓
Risk State Migration
     ↓
Preventive Intervention
     ↓
Collections & Recovery
     ↓
Feedback Loop
```

---

# System Architecture

```text
┌─────────────────────────────────────────────┐
│             LOAN ORIGINATION                │
└─────────────────────────────────────────────┘

Customer Application
            │
            ▼

┌──────────────────────────────────┐
│ Existing Underwriting System     │
└──────────────────────────────────┘

            │

Outputs:

Grade
Subgrade
Risk Tier
Approval Decision

            │
            ▼

┌──────────────────────────────────┐
│ Risk Anchor Creation             │
└──────────────────────────────────┘

Example:

Grade = B3

Historical Default Rate = 13.39%

Baseline PD = 13.39%

            │
            ▼

PostgreSQL Risk Store

═══════════════════════════════════════════════

      DYNAMIC RISK PORTFOLIO REBALANCER

═══════════════════════════════════════════════

            │
            ▼

┌──────────────────────────────────┐
│ Event Ingestion Layer            │
└──────────────────────────────────┘

Internal Sources:

• Credit Card Transactions
• Savings Accounts
• Checking Accounts
• EMI Repayment History

External Sources:

• Open Banking APIs
• Account Aggregator Networks

            │
            ▼

┌──────────────────────────────────┐
│ Kafka Event Stream               │
└──────────────────────────────────┘

Topic:

customer_events

            │
            ▼

┌──────────────────────────────────┐
│ Feature Aggregation Engine       │
└──────────────────────────────────┘

Produces Rolling Features:

• Credit Utilization
• Salary Consistency
• Payment Ratio
• Cash Withdrawal Frequency
• Delinquency Count
• Spending Velocity
• Balance Stability

            │
            ▼

┌──────────────────────────────────┐
│ Hidden State Engine              │
└──────────────────────────────────┘

States:

Healthy
Stress
Panic
Critical

Example:

Healthy → Stress
Stress → Panic
Panic → Critical

Implemented Using:

• Hidden Markov Model
or
• Markov State Transition Matrix

            │
            ▼

┌──────────────────────────────────┐
│ Dynamic PD Engine                │
└──────────────────────────────────┘

Input:

Baseline PD
+
Behavior Features
+
Current Hidden State

Output:

Current PD(t)

Example:

Origination PD = 13.39%

Current PD = 38.22%

            │
            ▼

┌──────────────────────────────────┐
│ Explainability Layer             │
└──────────────────────────────────┘

Using SHAP

Example:

PD = 38.22%

Drivers:

+ Salary Flatline
+ Utilization Spike
+ Cash Advance Activity

            │
            ▼

┌──────────────────────────────────┐
│ Risk Migration Engine            │
└──────────────────────────────────┘

Calculates:

Risk Drift Score

Risk Drift
=
Current PD
−
Origination PD

Example:

38.22% - 13.39%

=
24.83%

            │
            ▼

┌──────────────────────────────────┐
│ Intervention Recommendation      │
└──────────────────────────────────┘

Healthy
→ No Action

Stress
→ Reminder

Panic
→ Restructuring Offer

Critical
→ Collections Escalation

            │
            ▼

┌──────────────────────────────────┐
│ CRM / Recovery System            │
└──────────────────────────────────┘

Examples:

Salesforce

Collections Platform

Internal CRM

═══════════════════════════════════════════════

               FEEDBACK LOOP

═══════════════════════════════════════════════

Intervention
      │
      ▼

Customer Response

      │
      ▼

Recovered?
Defaulted?
Restructured?

      │
      ▼

Model Retraining

```

---

# LendingClub-Based MVP Architecture

Since transaction-level banking data is unavailable, the MVP will use LendingClub loan portfolio data.

## Step 1: Underwriting Anchor

Use:

```text
grade
sub_grade
```

Example:

```text
A → 6.04% default rate
B → 13.39% default rate
...
G → 49.93% default rate
```

These become baseline risk estimates.

---

## Step 2: PD Model

Train:

```text
XGBoost
```

Using:

```text
loan_amnt
annual_inc
dti
revol_util
purpose
grade
sub_grade
emp_length
fico
```

Target:

```text
loan_status
```

Where:

```python
default = (
    loan_status == "Charged Off"
)
```

Output:

```text
PD = 17.2%
```

---

## Step 3: State Transition Layer

Create latent borrower states:

```text
Healthy
Stress
Panic
Critical
```

Example Transition Matrix:

```text
Healthy

90% Stay Healthy
10% Move Stress

Stress

70% Stay Stress
20% Recover
10% Move Panic

Panic

60% Stay Panic
10% Recover
30% Critical
```

---

## Step 4: Behavioral Simulation Layer

Generate synthetic monitoring signals.

Healthy:

```text
Stable Salary
Low Utilization
Regular Payments
```

Stress:

```text
Salary Delays
Higher Utilization
Minimum Due Payments
```

Panic:

```text
Salary Missing
Cash Withdrawals
Near-Max Utilization
```

Critical:

```text
Missed Payments
Collection Events
```

---

## Step 5: Dynamic Risk Updating

Example:

```text
Month 0

PD = 13.39%

State = Healthy
```

```text
Month 1

PD = 16.2%

State = Healthy
```

```text
Month 2

PD = 24.1%

State = Stress
```

```text
Month 3

PD = 37.5%

State = Panic
```

```text
Month 4

PD = 58.9%

State = Critical
```

---

## Step 6: SHAP Explainability

Example:

```text
Current PD = 58.9%

Top Contributors:

+ Utilization Spike
+ Salary Gap
+ Missed Payment
```

---

## Step 7: Dashboard

Display:

```text
Customer ID

Grade

Subgrade

Origination PD

Current PD

Risk Drift

Current State

SHAP Explanations

Recommended Intervention
```

---

# Technology Stack

## Data Processing

```text
Python
Pandas
NumPy
SciPy
```

---

## Machine Learning

```text
Scikit-Learn
XGBoost
LightGBM
```

---

## Hidden State Modeling

```text
hmmlearn

or

Custom Markov Chains
```

---

## Explainability

```text
SHAP
```

---

## Storage

```text
PostgreSQL
```

---

## API Layer

```text
FastAPI
```

---

## Dashboard

```text
Streamlit
```

---

## Event Streaming (Enterprise)

```text
Kafka
Kafka Connect
Faust
Spark Streaming
```

---

## Containerization

```text
Docker
```

---

## Orchestration

```text
Kubernetes
```

---

# Key Innovation

Traditional Credit Systems:

```text
Application
     ↓
Underwriting
     ↓
Static Grade
     ↓
Default
```

Dynamic Risk Portfolio Rebalancer:

```text
Application
     ↓
Underwriting
     ↓
Baseline PD
     ↓
Behavior Monitoring
     ↓
Dynamic PD(t)
     ↓
Risk Drift Detection
     ↓
Preventive Intervention
     ↓
Collections
```

---

# Core KPI

```text
Risk Drift Score
=
Current PD − Origination PD
```

Example:

| Customer | Origination PD | Current PD | Drift |
|----------|----------------|------------|--------|
| A | 6.04% | 8.10% | +2.06% |
| B | 13.39% | 38.22% | +24.83% |
| C | 22.50% | 25.10% | +2.60% |

The platform's primary objective is to identify borrowers whose risk profile has materially deteriorated since underwriting and intervene before charge-off occurs.
