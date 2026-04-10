# Confusion Matrix

## Model Performance

|                | Predicted Phishing | Predicted Legitimate |
|----------------|-------------------|----------------------|
| **Actual Phishing**   | TP = 4            | FN = 1               |
| **Actual Legitimate** | TN = 4            | FP = 1               |

## Metrics

- **Accuracy:** 80%  
- **Precision:** 80%  
- **Recall:** 80%  
- **F1 Score:** 80%  

## Interpretation

The model correctly identified 4 phishing emails (true positives) and 4 legitimate emails (true negatives). However, it made 1 false negative (missed phishing email) and 1 false positive (incorrectly flagged legitimate email).

False negatives are more critical in this context because they allow real phishing attacks to go undetected, which could lead to security risks such as credential theft or unauthorized access.

Overall, the model shows balanced performance, but improvements should focus on reducing false negatives.
