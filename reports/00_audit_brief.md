# Audit Brief: PaySim Fraud Dataset

## Context
PaySim simulates a mobile money service based on real logs from an African country. Its fraud represents account takeover: fraudsters take control of customer accounts, transfer the funds out, then cash out. The dataset is widely used for fraud detection projects, often with near-perfect reported scores.

## Audit question
Can this dataset support an honest fraud detection model, one that relies on information available at the time of payment?

## Existing controls
| Control | Alerts per day | Precision | Fraud cases caught | Fraud value caught |
|---|---|---|---|---|
| `isFlaggedFraud` column | ~0.5 | 100% | 0.2% | 0.6% |
| Documented rule (TRANSFER > 200,000) | ~13,200 | 0.67% | 33.4% | 49.4% |

The `isFlaggedFraud` column does not match its own documented rule.

## Checks performed
- Class balance, rule performance, amount distribution
- Daily and hourly volume vs fraud
- Balance columns: post-transaction artifacts vs pre-transaction patterns
- Direct account linkage
- Whether a simple rule already separates fraud, and what it misses
- Point-in-time destination history (strictly earlier hours only), validated on a 300-row raw recount and on all 2,770,409 TRANSFER/CASH_OUT rows by an independent recomputation: 0 mismatches on all four features

## Conclusion: only a weak honest signal
| Finding | Evidence |
|---|---|
| One balance pattern separates almost all fraud | Amount equal to the sender's full balance: 97.8% of fraud cases, 87.7% of value, no false alarms. Confirms Visbeek et al. (2023) |
| The 10,000,000 amount explains most misses, not a detector | 145 of 179 misses; 8.95% precision on its own |
| Time columns are confounded | Overall volume collapses on some days and in the early hours while fraud stays near constant |
| Balance columns carry artifacts | Destination balances both zero for 49.6% of fraud vs 0.06% of legitimate |
| Destination history shows a modest predictive association | 63.2% of fraud destinations have no prior activity vs 15.1% of legitimate. In a preliminary test ranking TRANSFER/CASH_OUT transactions (top 500 alerts/day, days 13–16), a history-based score caught 140 of 1,068 frauds (13.1%) vs 101 (9.5%) for amount alone; steadier, but not better every day |

The near-perfect scores often reported on PaySim are best explained by the balance pattern. Point-in-time destination history shows a modest predictive association in this PaySim release. Its generalizability and underlying cause remain unverified. An honest experiment is possible with these features and a chronological holdout.

## Limitations
Rules and scores were found and evaluated on the same data; the stability check is within one simulation run; same-hour order is unknown; results apply to this Kaggle release of PaySim only.

## Data source
Lopez-Rojas, E. A., Elmir, A., & Axelsson, S. (2016). PaySim: A financial mobile money simulator for fraud detection. EMSS 2016.
