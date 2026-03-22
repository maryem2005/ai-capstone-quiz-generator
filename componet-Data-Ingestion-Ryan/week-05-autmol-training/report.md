# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** [Your Name]
**Date:** March 22, 2026
**Capstone Project:** Cybersecurity Threat Detection
**My Component:** Phishing & Threat Classification

## Part A: Teachable Machine Training

### Training Setup
- **Task:** Phishing vs Legitimate email screenshot classification
- **Training images per class:** 25
- **Test images per class:** 5
- **Total training time:** ~30 seconds

### Test Results

| # | Actual Class | Predicted Class | Confidence | Correct? |
|---|--------------|-----------------|------------|----------|
| 1 | Phishing | Phishing | 100% | ✅ |
| 2 | Phishing | Phishing | 100% | ✅ |
| 3 | Phishing | Phishing | 100% | ✅ |
| 4 | Phishing | Phishing | 100% | ✅ |
| 5 | Phishing | Phishing | 100% | ✅ |
| 6 | Legitimate | Phishing | 100% | ❌ |
| 7 | Legitimate | Legitimate | 79% | ✅ |
| 8 | Legitimate | Phishing | 100% | ❌ |
| 9 | Legitimate | Legitimate | 95% | ✅ |
| 10 | Legitimate | Legitimate | 95% | ✅ |

### Confusion Matrix

| | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 5 | FN = 0 |
| **Actual: Legitimate** | FP = 2 | TN = 3 |

### Calculated Metrics
- **Accuracy:** 80%
- **Precision:** 71.4%
- **Recall:** 100%
- **F1 Score:** 83.3%

### Interpretation
The model achieved perfect recall, correctly identifying all 5 phishing 
emails, but flagged 2 legitimate emails as phishing (false positives), 
resulting in 71.4% precision. For cybersecurity, this tradeoff is 
acceptable missing a phishing email is far more dangerous than a 
false alarm. Model improvement would come from adding more diverse 
legitimate email training images, particularly those containing 
links and urgent-looking formatting that visually resemble phishing.

---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested
1. **Generic:** distilbert-base-uncased-finetuned-sst-2-english (sentiment)
2. **Fine-Tuned A:** ealvaradob/bert-finetuned-phishing — BERT large 
fine-tuned specifically for phishing detection across emails, URLs, 
SMS, and websites
3. **Fine-Tuned B:** mrm8488/bert-tiny-finetuned-sms-spam-detection — 
BERT tiny fine-tuned for SMS spam detection, classifies as SPAM or HAM

### Results

| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Score | Best Model |
|-------|-----------------------|----------------------------|--------------------|------------|
| Unauthorized login from Moscow | NEGATIVE (0.9961) | phishing (0.9996) | 0.7045 | Fine-Tuned A |
| Routine firewall update | NEGATIVE (0.9986) | benign (1.0000) | 0.5376 | Fine-Tuned A |
| Phishing email spoofed Amazon | NEGATIVE (0.9959) | phishing (1.0000) | 0.7415 | Fine-Tuned A |
| Multiple failed SSH attempts | NEGATIVE (0.9994) | phishing (0.9752) | 0.5719 | Fine-Tuned A |
| System utilization normal | NEGATIVE (0.9880) | benign (0.9996) | 0.8388 | Fine-Tuned A |

### Analysis

**Generic model strengths:** The generic sentiment model was consistently 
confident in its scores, showing high certainty across all 5 inputs. 
It processed inputs quickly and reliably without errors.

**Generic model weaknesses:** It labeled every single input as NEGATIVE 
regardless of whether it was a real threat or routine activity. It has 
no concept of phishing or cybersecurity it only understands 
positive/negative sentiment, making it completely unsuitable for 
threat detection.

**Fine-tuned model advantage:** Fine-Tuned A (bert-finetuned-phishing) 
clearly outperformed the generic model by correctly distinguishing 
between phishing threats and benign activity. It correctly flagged 
records 1, 3, and 4 as phishing and correctly identified records 2 
and 5 as benign exactly the expected results for a cybersecurity 
classifier.

**Biggest surprise:** The generic model labeled the obvious phishing 
email (record 3 — "Phishing email with spoofed Amazon domain") as 
NEGATIVE with 99.59% confidence, showing that sentiment models have 
zero awareness of security context even when the word "phishing" 
appears explicitly in the text.

### Recommended Model for My Capstone Component
**Component:** Phishing & Threat Classification
**Primary model:** ealvaradob/bert-finetuned-phishing — this model was 
trained specifically on phishing datasets covering emails, URLs, and 
SMS, making it directly relevant to cybersecurity threat detection.
**Confidence threshold:** 0.90 — predictions above 90% confidence 
trigger automatic flagging; below 90% route to human analyst review.
**Priority metric:** Recall — in cybersecurity it is more costly to 
miss a real threat (false negative) than to flag a legitimate item 
for review (false positive), so maximizing recall is the priority.

---

## Limitations & Next Steps
With only 10 test images for the Teachable Machine model and 5 text 
inputs for the fine-tuned comparison, these results are preliminary 
estimates rather than definitive measurements. With more time, I would 
expand the test dataset to 50+ inputs covering edge cases such as 
ambiguous language, very short inputs, and mixed signals. I would also 
explore fine-tuning my own BERT model on a domain-specific cybersecurity 
dataset using Hugging Face AutoTrain to improve performance on 
network-level threats like the SSH brute force attempt, which 
bert-finetuned-phishing misclassified as benign.
