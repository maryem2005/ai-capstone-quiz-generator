# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** Maria Shirin
---
**Date:** April 10, 2026
---
**Capstone Project:** AI Capstone Quiz Generator
---
**My Component:** Integration (AI Workflow & Model Evaluation)

---

## Part A: Teachable Machine Training

### Training Setup

* **Task:** Phishing vs Legitimate Email Classification
* **Training images per class:** 20
* **Test images per class:** 5
* **Total training time:** ~30 seconds

---

### Test Results

| #  | Actual Class | Predicted Class | Confidence | Correct? |
| -- | ------------ | --------------- | ---------- | -------- |
| 1  | Phishing     | Phishing        | 100%       | Yes      |
| 2  | Phishing     | Phishing        | 96%        | Yes      |
| 3  | Phishing     | Phishing        | 99%        | Yes      |
| 4  | Phishing     | Legitimate      | 94%        | No       |
| 5  | Phishing     | Phishing        | 51%        | Yes      |
| 6  | Legitimate   | Legitimate      | 99%        | Yes      |
| 7  | Legitimate   | Legitimate      | 100%       | Yes      |
| 8  | Legitimate   | Legitimate      | 100%       | Yes      |
| 9  | Legitimate   | Legitimate      | 100%       | Yes      |
| 10 | Legitimate   | Phishing        | 100%       | No       |

---

## Confusion Matrix & Metrics

### Confusion Matrix

|                       | Predicted Phishing | Predicted Legitimate |
| --------------------- | ------------------ | -------------------- |
| **Actual Phishing**   | TP = 4             | FN = 1               |
| **Actual Legitimate** | TN = 4             | FP = 1               |

---

### Performance Metrics

* **Accuracy:** 80%
* **Precision:** 80%
* **Recall:** 80%
* **F1 Score:** 80%

---

## Interpertation
My model performs equally in precision and recall, both at 80%, meaning it is balanced in identifying phishing emails and minimizing false alarms. 
It made both false negative errors (missing a phishing email) and false positive errors (flagging a legitimate email), but false negatives are more serious because they allow real phishing attacks to go undetected, potentially leading to credential theft, financial loss, or system compromise. 
The model could be improved by increasing the amount and diversity of training data to help it better distinguish between phishing and legitimate patterns.


## Part B: Generic vs Fine-Tuned Model Comparison
### Models Tested
1. **Generic:** distilbert-base-uncased-finetuned-sst-2-english (sentiment)
 Fine-tuned Model A: mrm8488/bert-tiny-finetuned-sms-spam-detection
A text classification model fine-tuned to classify messages as spam or not spam, making it useful for detecting phishing or malicious communications in a cybersecurity context.
 Fine-tuned Model B: cardiffnlp/twitter-roberta-base-sentiment-latest
A sentiment analysis model that classifies text as positive, neutral, or negative, useful for analyzing the tone and intent of messages in text data.


---

### Results
| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Label (Score) | Best Model |
|-------|-----------------------|-----------------------------|----------------------------|------------|
| Unauthorized login from IP... | NEGATIVE (0.9966) | LABEL_0 (0.7415) | negative (0.5222) | Generic |
| Routine firewall update... | NEGATIVE (0.9986) | LABEL_1 (0.5719) | negative (0.8473) | Generic |
| Phishing email with spoofed... | NEGATIVE (0.9959) | LABEL_0 (0.6224) | neutral (0.8358) | Generic |
| Multiple failed SSH attempts... | NEGATIVE (0.9994) | LABEL_1 (0.5376) | neutral (0.8743) | Generic |
| System resource utilization... | — | LABEL_0 (0.8388) | — | Fine-Tuned A |

---
## Analysis

**Generic model strengths**:
The generic model performed very well across most records, consistently giving high confidence scores (around 0.99) and correctly identifying cybersecurity-related threats like phishing and suspicious activity. 
It was especially strong at detecting clearly malicious patterns, making it reliable for general classification tasks.

**Generic model weaknesses**:
The generic model struggled with more neutral or non-threatening inputs, such as normal system activity. It tended to classify everything as negative, even when there were no real threats, showing a lack of nuance and potential overclassification of risk.

**Fine-tuned model advantage**:
The fine-tuned models provided more specialized insights. Fine-Tuned A was better at distinguishing between categories (LABEL_0 vs LABEL_1), while Fine-Tuned B added useful context by analyzing sentiment (negative vs neutral). 
In cases like system activity or less obvious threats, these models offered more variation and nuance compared to the generic model.

**Biggest surprise**:
The biggest surprise was how confident the generic model was even when it may not have been fully accurate. 
It consistently gave very high confidence scores, while the fine-tuned models were sometimes less confident but more nuanced. This showed that higher confidence does not always mean better or more accurate results.

---

### Recommended Model for My Capstone Component
**Component**: AI Quiz Generator — Classification & Alert Analysis

**Primary model**: mrm8488/bert-tiny-finetuned-sms-spam-detection: 
This model is best suited for the task because it is specifically fine-tuned for spam detection, which closely aligns with identifying phishing and malicious messages in cybersecurity alerts. 
It provides more task-relevant classification compared to a general-purpose model.

**Confidence threshold**: 85%
A threshold of 85% ensures that only high-confidence predictions are automatically accepted, while lower-confidence results are flagged for review.
This is important because the model can sometimes be confidently wrong, so adding a threshold reduces the risk of incorrect classifications.

**Priority metric**: Recall — Recall is the most important metric for this project because missing a phishing or malicious alert (false negative) is more dangerous than incorrectly flagging a safe one. 
In a cybersecurity context, it is better to catch as many potential threats as possible, even if it results in some false positives.

## Limitations & Next Steps
With more time and data, I would improve the model by expanding the dataset and increasing its diversity. This includes adding more examples of phishing emails, legitimate messages, and edge cases such as ambiguous or borderline alerts. More varied data would help the model generalize better and reduce overfitting to specific patterns.
I would also consider fine-tuning my own model specifically for cybersecurity alert classification. Instead of relying on a general spam detection model, training a model on domain-specific data (e.g., security logs, phishing reports, incident alerts) would likely improve accuracy and make predictions more relevant to real-world scenarios.
Additionally, I would test more advanced and specialized models. For example, I could experiment with larger transformer models or models trained specifically on security datasets, as well as compare performance with ensemble methods (combining multiple models). I would also explore models designed for anomaly detection to better identify unusual or suspicious behavior in alerts.
