# Week 5: AutoML & No-Code Model Training

Trained a custom image classifier with Google Teachable Machine and compared generic vs fine-tuned Hugging Face models for the AI Quiz Generator component of our AI Capstone Quiz Generator project.

## Custom Model Training
- Built a Phishing/Legitimate image classifier with Teachable Machine  
- Achieved 80% accuracy on 10 held-out test images  
- Precision: 80% | Recall: 80% | F1: 80%  

## Fine-Tuned Model Comparison
Compared 3 models (1 generic + 2 fine-tuned) on 5 test inputs:
- Generic: distilbert-sst-2 (sentiment)  
- Fine-Tuned A: mrm8488/bert-tiny-finetuned-sms-spam-detection  
- Fine-Tuned B: cardiffnlp/twitter-roberta-base-sentiment-latest  

## Finding
Recommended mrm8488/bert-tiny-finetuned-sms-spam-detection for the AI Quiz Generator component because it is specifically trained for spam detection, making it more relevant for identifying phishing and malicious messages. Fine-tuned models showed comparable performance with more relevant labels and better alignment with the cybersecurity domain.
