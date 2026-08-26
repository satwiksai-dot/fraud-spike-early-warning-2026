# Fraud Spike Early-Warning Model

An early-warning machine learning system for identifying transaction windows that are at elevated risk of a future fraud spike.

## Problem

Fraud does not always appear as isolated fraudulent transactions. In some cases, fraudulent activity can increase sharply within a short period.

The goal of this project is to identify these potential fraud-spike windows early, using transaction activity observed before the spike.

The model is designed as a **ranking system**, helping prioritize which windows should be investigated first rather than treating every window as equally risky.

## Approach

The project uses transaction-level data from the IEEE-CIS Fraud Detection dataset and converts transactions into short time windows.

For each window, features are calculated from early transaction activity, including:

- Number of early transactions
- Number of unique cards
- Transaction count relative to typical activity
- Maximum transactions within one minute
- Transactions per card
- Early total transaction amount
- Transaction amount relative to typical activity

A Random Forest classifier is then used to produce a risk score for each window.

## Final Model

The final model uses 7 features:

1. `early_unique_cards`
2. `early_transaction_count`
3. `count_vs_typical`
4. `max_1m_transactions`
5. `transactions_per_card`
6. `early_total_amount`
7. `amount_vs_typical`

## Results

The problem is highly imbalanced, with fraud-spike windows representing only **0.571%** of the test set.

### Model Performance

| Metric | Result |
|---|---:|
| Validation PR-AUC | 0.011987 |
| Test PR-AUC | 0.016364 |
| Test spike prevalence | 0.571% |

### Top-K Alert Performance

| Alert group | Alerts | Real spikes | Precision | Recall |
|---|---:|---:|---:|---:|
| Top 0.5% | 18 | 0 | 0.0% | 0.0% |
| Top 1% | 36 | 0 | 0.0% | 0.0% |
| Top 2% | 73 | 2 | 2.7% | 9.5% |
| Top 5% | 183 | 4 | 2.2% | 19.0% |
| Top 10% | 367 | 6 | 1.6% | 28.6% |

The model is therefore more useful as a **risk-ranking mechanism** than as a high-precision binary fraud detector.

## Incident Grouping

At the top 5% alert threshold:

- 183 raw alerts were generated
- These were grouped into 140 incidents using a 30-minute grouping window
- 4 incidents contained a real fraud spike
- Incident precision was 2.86%
- Average alerts per incident was 1.31

Grouping nearby alerts helps reduce repeated alerts around the same underlying activity.

## Feature Importance

The most important features in the final model were:

| Feature | Importance |
|---|---:|
| `early_unique_cards` | 19.73% |
| `early_transaction_count` | 16.02% |
| `count_vs_typical` | 15.68% |
| `max_1m_transactions` | 14.30% |
| `transactions_per_card` | 12.15% |
| `early_total_amount` | 11.44% |
| `amount_vs_typical` | 10.68% |

This suggests that early transaction volume, card diversity, deviations from typical activity, and short-term transaction bursts are the strongest signals among the selected features.

## Limitations

The signal remains weak because the target is extremely imbalanced.

The model can rank some future fraud-spike windows above the base rate, but it does not provide high precision at very small alert volumes.

Further improvement would likely require additional transaction-level information or richer temporal features.

## Repository Contents

- `fraud_spike_early_warning.ipynb` - complete analysis, feature engineering, model development and evaluation

## Conclusion

This project explores whether fraud-spike activity can be detected before the spike occurs.

The final model provides a weak but measurable early-warning signal and is best interpreted as a prioritization tool for investigation rather than an automated fraud decision system.
