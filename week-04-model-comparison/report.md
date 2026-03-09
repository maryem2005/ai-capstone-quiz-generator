# Model Comparison Report – Week 4
Name: Maria Shirin  
Date:March 9, 2026  
Capstone Project: Security Alert Analysis Pipeline  
My Component: Multi-Model Threat Classification Workflow

# Test Setup

Input dataset: 5 cybersecurity alert records covering:
- Suspicious login attempts
- Routine firewall maintenance
- Phishing email detection
- SSH authentication failures
- Normal system monitoring activity

The dataset intentionally included:
- 2 clearly concerning/high-severity alerts
- 1 ambiguous alert
- 2 routine/benign alerts

# Models Tested

1. distilbert-base-uncased-finetuned-sst-2-english
   Used for sentiment analysis.

2. facebook/bart-large-mnli  
   Used for zero-shot classification to categorize alerts.

3. dslim/bert-large-NER
   Used for named entity recognition.

4. Groq Llama 3 8B  
   Used for LLM-based security classification and explanation.

# Evaluation Criteria
The models were compared based on:

- Label accuracy
- Confidence scores
- Quality of explanations
- Speed
- Ease of integration in n8n

# Results Summary

| Record | Sentiment | Zero-Shot | NER Entities | Groq |
|------|------|------|------|------|
| 1 | NEGATIVE | possible anomaly | None | MEDIUM |
| 2 | NEGATIVE | routine activity | None | INFORMATIONAL |
| 3 | NEGATIVE | possible anomaly | None | MEDIUM|
| 4 | NEGATIVE | possible anomaly | None | CRITICAL |
| 5 | NEGATIVE | routine activity | None | INFORMATIONAL|

# Analysis
## Where models agreed
The models generally agreed on which records were suspicious versus routine.

Records 1 and 3 were consistently identified as concerning:
- Sentiment labeled them as negative.
- Zero-shot classified them as possible anomalies.
- Groq classified them as MEDIUM or HIGH severity.

Records 2 and 5 were recognized as routine activity by the zero-shot model and Groq, indicating maintenance or normal system behavior.

## Where models disagreed
The main disagreement occurred with the sentiment model, which labeled every alert as NEGATIVE, including routine events.  
This happens because cybersecurity alerts often contain technical or warning-related language that appears negative even when the event is harmless.

Another issue occurred with the Groq LLM, which returned no response for records 4 and 5 during execution. This likely resulted from API response inconsistencies.

## Most accurate model overall
The Zero-Shot classification model (facebook/bart-large-mnli) produced the most sensible results overall.

It correctly identified:
- suspicious security events as possible anomalies
- routine maintenance alerts as routine activity

This made it the most reliable model for distinguishing between benign and concerning alerts.

## Fastest / most practical model

The sentiment model (distilbert) was the fastest and easiest to integrate.  
However, it was less useful for cybersecurity classification because it interpreted most alerts as negative regardless of severity.


# Recommended Models for My Capstone Component

Component: Security Alert Classification Pipeline

Primary model: facebook/bart-large-mnli  
This model is best suited for categorizing alerts into meaningful categories such as routine activity or anomalies without requiring labeled training data.

Secondary model: Groq Llama 3 8B  
The LLM provides useful explanations and contextual reasoning about alerts, helping analysts understand why an event may be concerning.

Rejected models and why:

- distilbert sentiment model:  
  Too simplistic for cybersecurity logs and incorrectly labels routine alerts as negative.

- dslim NER model: 
  Not effective for cybersecurity alerts because most entities (IP addresses, hostnames, system identifiers) are not recognized by general-purpose NER models.

# Failure Cases and Limitations

The NER model behaved differently from the other models because it returned variable-length outputs depending on how many named entities were detected in each alert. In some cases, the records did not contain strong enough named entities, and in others, the output structure made one-to-one Airtable updates harder to align than the sentiment, zero-shot, and Groq branches. Even with the updated instructions to align the words more with Ner, Ner was still having issues populating itself. As a result, the NER field was left unpopulated/None for stability while the other three model comparisons were completed successfully.

Additionally, the Groq LLM returned empty responses for two records. This highlights a potential limitation when relying on external APIs, where responses may occasionally fail or return incomplete results.

# Next Steps

If more time were available, the following improvements could be explored:

- Testing additional cybersecurity-specific NER models capable of detecting IP addresses, domains, and system identifiers.
- Expanding the dataset to include more diverse alert types.
- Adding more classification labels to improve anomaly detection.
- Comparing additional LLMs for better explanation quality.
