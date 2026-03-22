# Confusion Matrix — Phishing vs Legitimate Classifier

## Test Results Summary
- **Model:** Google Teachable Machine (custom trained)
- **Test images:** 10 (5 phishing, 5 legitimate)
- **Date:** March 22, 2026

## Confusion Matrix

|  | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 5 | FN = 0 |
| **Actual: Legitimate** | FP = 2 | TN = 3 |

## Calculated Metrics

| Metric | Formula | Result |
|---|---|---|
| Accuracy | (TP+TN) / (TP+TN+FP+FN) | 80% |
| Precision | TP / (TP+FP) | 71.4% |
| Recall | TP / (TP+FN) | 100% |
| F1 Score | 2 × (P×R) / (P+R) | 83.3% |

## Notes
Model has perfect recall but lower precision — correctly caught 
all phishing emails but flagged 2 legitimate emails as phishing.
