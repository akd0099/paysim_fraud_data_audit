# Project Charter: Transaction Fraud Detection

## 1. Business problem
A mobile money service (simulated, based on real logs from an African country) is losing money to account-takeover fraud. Fraudsters take control of customer accounts, transfer the funds to another account, then cash out of the system. Customers lose their balances, and the operator bears reimbursement costs and loss of trust. These frauds are hard to spot because each step looks like a normal transfer or cash-out.

## 2. Current state
A rule flags any single transfer above 200,000 (`isFlaggedFraud`). How much fraud this rule catches is not yet known and will be measured during EDA. Fraud below the threshold is not flagged at all.

Two controls exist. The `isFlaggedFraud` flag raised 16 alerts in 30 days (100% precision) but caught only 0.6% of fraud value, and does not match its documented rule. The documented rule (any TRANSFER above 200,000) would catch 49% of fraud value but generate ~13,200 alerts a day at 0.7% precision, beyond any review team's capacity. Fraud below 200,000 is not caught by either.


## 3. Objective
Score each TRANSFER and CASH_OUT transaction by fraud risk, so that a review team with fixed daily capacity catches more fraud value than the current 200,000 rule. Generate a written case summary for each alert to speed up investigation.

## 4. Constraints
- **Alert budget:** the review team can investigate 500 alerts per day (to be validated against fraud volume in EDA).
- **Cost asymmetry:** a missed fraud costs more than a false alarm.
- **Data:** balance columns (`oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`) are excluded from modelling. Per the dataset author, fraudulent transactions are cancelled, so these columns leak the label.
- **Currency:** amounts are in local currency units.
- **Tools:** free and open-source only; no paid APIs.
- `step`/`day` are not used as features: legitimate volume varies by day and hour while fraud is injected at a near-constant rate, so time features partly encode the simulator rather than fraud behaviour. Hour of day is tested with and without in phase 4.

## 5. Success metrics
- **Primary:** share of total fraud value caught within the daily alert budget, compared to the 200,000 rule.
- **Secondary:** PR-AUC; precision of alerts; share of fraud count caught.
- **LLM layer:** case summary quality, scored against a rubric defined in phase 6.

## 6. Scope
**In scope**
- TRANSFER and CASH_OUT transactions (the only types containing fraud)
- Offline batch scoring with a time-based train/validation/test split
- Unsupervised (Isolation Forest) and supervised (XGBoost) models, compared
- LLM-generated case summaries and their evaluation

**Out of scope**
- Real-time deployment
- CASH_IN, DEBIT and PAYMENT transactions
- Balance columns as model features

## 7. Deliverables
- EDA and modelling notebooks
- Feature engineering and scoring pipeline (`src/`)
- Model comparison table against the 200,000 rule
- LLM investigation layer with evaluation results
- Dashboard of alerts and performance
- README with problem, approach, results and limitations

## Data source
Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. EMSS 2016. Dataset via Kaggle.