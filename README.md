# PaySim Fraud Dataset: Data Audit

An audit of the PaySim synthetic mobile money dataset to check whether it can support a fraud detection model built on information available at the time of payment.

**Conclusion:** it cannot. After removing leakage and simulator artifacts, two hard-coded simulator rules explain almost all fraud, so high model scores on this dataset largely reflect how the simulator generates fraud rather than learned fraud behaviour.

## Key findings

| # | Finding | Evidence |
|---|---|---|
| 1 | One rule explains almost all fraud | Flagging transactions where the amount equals the sender's full balance catches 97.8% of fraud cases and 87.7% of fraud value at 99.96% precision (~260 alerts/day, 3 false alarms in 30 days). Only ~0.0001% of legitimate TRANSFER and CASH_OUT transactions match |
| 1b | Most remaining fraud hits a simulator cap | 145 of the 179 frauds missed by the rule above (81%) are exactly 10,000,000, the simulator's per-transaction cap, where large balances are emptied in several transfers |
| 2 | Existing controls are precise but blind, or broad but unworkable | `isFlaggedFraud`: 16 alerts in 30 days, 0.6% of fraud value caught. Documented rule (TRANSFER > 200,000): ~13,200 alerts/day at 0.7% precision, 49% of fraud value caught |
| 3 | The `isFlaggedFraud` column does not match its documentation | 409,110 transfers exceed 200,000; only 16 are flagged |
| 4 | Legitimate volume collapses on some days; fraud does not | Days 0–1, 5–16: 350k–570k transactions/day. Days 2–4, 17–30: 6k–58k. Fraud stays at ~220–316/day throughout. Day 30 is 100% fraud |
| 5 | Hour of day is mostly a simulator artifact | Legitimate activity is near zero at night, so the fraud rate reaches ~22% at hours 4–5 |
| 6 | Balance columns leak the label | Stated by the dataset author. Destination balances are both zero for 49.6% of fraud vs 0.06% of legitimate transactions |
| 7 | The fraud chain cannot be traced | 0 fraud transfer destinations later originate a fraud cash-out; 99.85% of senders appear only once |

## Rule comparison

| Rule | Alerts/day | Precision | Fraud value caught |
|---|---|---|---|
| `isFlaggedFraud` column | ~0.5 | 100% | 0.6% |
| Documented rule: TRANSFER > 200,000 | ~13,200 | 0.7% | 49% |
| Amount equals full sender balance | ~260 | 99.96% | 87.7% |

## What this means for modelling

| Feature group | Status |
|---|---|
| Post-transaction balances | Leakage |
| Pre-transaction balance | Encodes the simulator's fraud rule |
| `step`, `day`, `hour` | Simulator volume artifacts |
| Account IDs | No usable history or linkage |
| `amount`, `type` | Valid but weak |

## Repository

| Path | Contents |
|---|---|
| `notebooks/01_eda.ipynb` | Full audit with code, outputs and findings |
| `reports/00_audit_brief.md` | One-page summary: audit question, checks, decision |

## Reproduce

1. Download the dataset from Kaggle ("Synthetic Financial Datasets For Fraud Detection") and save it as `data/raw/paysim.csv`
2. `python -m venv .venv` and activate it
3. `pip install -r requirements.txt`
4. Run `notebooks/01_eda.ipynb`

## Data source

Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. EMSS 2016.
