# Week 5: AutoML & No-Code Model Training

Trained a custom image classifier with Google Teachable Machine and 
compared generic vs fine-tuned Hugging Face models for the Phishing 
& Threat Classification component of our Cybersecurity Threat 
Detection capstone project.

## Custom Model Training
- Built a Phishing/Legitimate image classifier with Teachable Machine
- Achieved 80% accuracy on 10 held-out test images
- Precision: 71.4% | Recall: 100% | F1: 83.3%

## Fine-Tuned Model Comparison
Compared 3 models (1 generic + 2 fine-tuned) on 5 test inputs:
- Generic: distilbert-sst-2 (sentiment)
- Fine-Tuned A: ealvaradob/bert-finetuned-phishing
- Fine-Tuned B: mrm8488/bert-tiny-finetuned-sms-spam-detection

## Finding
Recommended ealvaradob/bert-finetuned-phishing for Phishing & Threat 
Classification because it was trained specifically on phishing datasets 
covering emails, URLs, and SMS messages, making it directly relevant 
to cybersecurity threat detection. Fine-tuned models showed higher 
performance with more relevant labels — correctly distinguishing 
phishing from benign activity where the generic model only returned 
undifferentiated NEGATIVE sentiment scores.

See `report.md` for full analysis.
