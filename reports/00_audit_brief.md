# Audit Brief: PaySim Fraud Dataset

## Context
PaySim simulates a mobile money service based on real logs from an African country. Its fraud represents account takeover: fraudsters take control of customer accounts, transfer the funds out, then cash out. The dataset is widely used for fraud detection projects, often with near-perfect reported scores.

## Audit question
Can this dataset support an honest fraud detection model, one that relies on information available at the time of payment?

## Existing controls
| Control | Alerts per day | Precision | Fraud value caught |
|---|---|---|---|
| `isFlaggedFraud` column | ~0.5 | 100% | 0.6% |
| Documented rule (TRANSFER > 200,000) | ~13,200 | 0.7% | 49% |

The `isFlaggedFraud` column does not match its own documented rule.

## Checks performed
- Class balance by transaction type
- Performance of existing rules
- Daily and hourly volume vs fraud patterns
- Leakage in balance columns
- Traceability of fraud across accounts
- Whether a simple rule already separates fraud

## Decision: not proceeding to modelling
| Reason | Evidence |
|---|---|
| One rule explains almost all fraud | Amount equal to the sender's full balance: 87.7% of fraud value caught at 99.96% precision. The simulator encodes fraud as "empty the account". Most of the remaining fraud is exactly 10,000,000, the simulator's per-transaction cap |
| Remaining features carry artifacts | Legitimate volume collapses on some days and at night while fraud stays constant, so time features encode the simulator |
| Balance columns leak the label | Stated by the dataset author; confirmed in destination balances |
| No usable account history | 99.85% of senders appear once; fraud chains cannot be traced |

A model trained on this data would either rediscover the simulator's fraud rules or have too little honest signal to be meaningful.

## Data source
Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. EMSS 2016.
