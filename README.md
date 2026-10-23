# PaySim Fraud Dataset: Data Audit

An audit of the PaySim synthetic mobile money dataset to check whether it can support a fraud detection model built on information available at the time of payment.

**Conclusion:** only weakly. Almost all fraud follows a simulator pattern in the sender's balance, and several other columns carry simulator artifacts. Once those are excluded, point-in-time destination history shows a modest predictive association in this PaySim release. Its generalizability and underlying cause remain unverified. The near-perfect scores often reported on this dataset are best explained by the balance pattern rather than learned fraud behaviour.

## Key findings

| # | Finding | Evidence |
|---|---|---|
| 1 | One balance pattern separates almost all fraud | Flagging TRANSFER/CASH_OUT where the amount exactly equals the sender's full balance catches 97.8% of fraud cases and 87.7% of fraud value with no false alarms (8,034 alerts, ~260/day). This confirms an observation reported by Visbeek et al. (2023) |
| 2 | The 10,000,000 amount explains most misses but is not a detector | 145 of the 179 missed frauds are exactly 10,000,000, all from accounts holding more than that; no fraud exceeds 10,000,000. On its own, flagging 10,000,000 has 8.95% precision (2,920 legitimate transfers share the amount). Combined with the balance pattern: 99.6% of cases, 99.8% of value, 73.7% precision, ~358 alerts/day; 34 frauds (0.4%) are still missed |
| 3 | Existing controls are precise but blind, or broad but unworkable | `isFlaggedFraud`: 16 alerts in 31 days, 0.2% of fraud cases, 0.6% of fraud value. Documented rule (TRANSFER > 200,000): ~13,200 alerts/day at 0.67% precision, 33.4% of fraud cases, 49.4% of fraud value |
| 4 | The `isFlaggedFraud` column does not match its documentation | 409,110 transfers exceed 200,000; only 16 are flagged |
| 5 | Overall volume collapses on some days and hours; fraud does not | Days 0–1 and 5–16: 350k–571k transactions/day; days 2–4 and 17–29: 6.7k–59k. Fraud stays at 220–316/day. Day 30 is 282 transactions, all fraud. Volume also drops to near zero in the early hours, so the hourly fraud rate spikes there |
| 6 | Balance columns: two separate issues | Post-transaction: destination balances both zero for 49.6% of fraud vs 0.06% of legitimate; origin balance consistent with the amount for 99.5% of fraud vs 9.9% of legitimate. Pre-transaction: available at payment time, but encodes the pattern in finding 1. The dataset author advises against all four |
| 7 | No direct transfer-to-cash-out link | No destination of a fraudulent TRANSFER appears as the origin of a fraudulent CASH_OUT. Sender history is highly limited: the number of unique sender IDs is 99.85% of the transaction count, indicating little repetition relative to the dataset's size. Destinations repeat far more (the number of unique destination IDs is 42.8% of the transaction count). Chains through legitimate transactions were not tested |
| 8 | Point-in-time destination history: modest predictive association | Using only earlier transactions, 63.2% of fraud destinations have no prior activity vs 15.1% of legitimate ones. In a preliminary, untuned test on days 13–16, TRANSFER and CASH_OUT transactions were ranked and the top 500 alerts per day evaluated: the history score caught 140 of 1,068 frauds (13.1%, ~7% precision) vs 101 (9.5%) for ranking by amount alone. Steadier than amount ranking, but not better every day. Generalizability and underlying cause remain unverified |

## Rule comparison

| Rule | Alerts/day | Precision | Fraud cases caught | Fraud value caught |
|---|---|---|---|---|
| `isFlaggedFraud` column | ~0.5 | 100% | 0.2% | 0.6% |
| Documented rule: TRANSFER > 200,000 | ~13,200 | 0.67% | 33.4% | 49.4% |
| Amount equals full sender balance | ~260 | 100% | 97.8% | 87.7% |
| Amount = 10,000,000 (standalone) | ~104 | 8.95% | 3.5% | 23.8% |
| Full balance OR 10,000,000 | ~358 | 73.7% | 99.6% | 99.8% |

## What this means for modelling

| Feature group | Status |
|---|---|
| Post-transaction balances | Strongly tied to the label through simulator artifacts; excluded |
| Pre-transaction sender balance | Available at payment time, but encodes a near-deterministic simulator pattern |
| `step`, `day`, `hour` | Heavily confounded by the simulator's volume pattern; excluded from a baseline unless validated on a chronological holdout |
| Sender history | Highly limited: unique sender IDs are 99.85% of the transaction count, indicating little repetition |
| Destination history (point-in-time) | Modest predictive association in this PaySim release; generalizability and underlying cause unverified |
| `amount`, `type` | Valid but weak on their own |

## Feature validation

The point-in-time destination features (`prior_in`, `prior_out`, `prior_in_amount`, `hours_since_prev`) use only transactions from strictly earlier hours. They were checked two ways:

- **Sample recount:** 300 transactions (150 fraud, 150 legitimate) recounted directly from the raw rows: 0 mismatches on all four features.
- **Full recomputation:** all 2,770,409 TRANSFER and CASH_OUT rows recomputed with a second, independent method (binary search over raw transactions sorted by account and step, with amounts summed in integer cents): 0 mismatches on all four features.

## Limitations

- The rules and the history score were identified on the same data they are evaluated on. The half-month comparison (98.1% and 97.5% of fraud caught, 100% precision in both) shows the balance pattern is stable within this run, not that it generalises.
- An independent test would need a separate PaySim run.
- PaySim does not record the order of transactions within the same hour, so point-in-time features use strictly earlier hours only.
- Results apply to this Kaggle release of PaySim.

## Repository

| Path | Contents |
|---|---|
| `notebooks/01_eda.ipynb` | Full audit with code, outputs and findings |
| `reports/00_audit_brief.md` | One-page summary: audit question, checks, conclusion |

## Reproduce

1. Download the dataset from Kaggle ("Synthetic Financial Datasets For Fraud Detection") and save it as `data/raw/paysim.csv`
2. `python -m venv .venv` and activate it
3. `pip install -r requirements.txt`
4. Open `notebooks/01_eda.ipynb`, restart the kernel and run all cells. Section 10 builds point-in-time features over 6.3M rows and needs several GB of memory

## Related work

Visbeek, S., Acar, E., & den Hengst, F. (2023). *Explainable Fraud Detection with Deep Symbolic Classification*. arXiv:2312.00586. Notes that predicting fraud when the amount equals the sender's balance gives very high accuracy, and drops `isFlaggedFraud` because it does not match its description. This audit confirms both and adds operational metrics (alerts per day, fraud value caught), the 10,000,000 amount, the volume artifacts and a validated point-in-time test of destination history.

## Data source

Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. EMSS 2016.
