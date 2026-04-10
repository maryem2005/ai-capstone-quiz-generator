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

## Analysis
### Where models agreed
The models generally agreed on which alerts represented routine system behavior versus potentially concerning activity. 
Records 2 and 5 were classified as routine or informational events by both the Zero-Shot model and the Groq LLM, indicating agreement that these alerts did not represent serious security issues.

### Where models disagreed
The Sentiment model labeled all records as negative because it was trained for general sentiment analysis rather than cybersecurity event classification. 
In contrast, the Zero-Shot model and Groq LLM were able to distinguish between routine activity and possible anomalies. 
Groq also identified Record 4 as CRITICAL, while the Zero-Shot model only labeled it as a possible anomaly, showing that the LLM applied stronger contextual reasoning.

## Most accurate model overall
The Groq LLM (Llama 3.1 8B) produced the most meaningful classifications overall.
Unlike the other models, it assigned clear severity levels such as MEDIUM, INFORMATIONAL, and CRITICAL based on the context of each alert.  
For example, it identified the repeated failed SSH authentication attempts as CRITICAL, while labeling routine maintenance events as INFORMATIONAL.

This demonstrated stronger contextual reasoning compared to the other models, which mostly categorized alerts as either routine activity or possible anomalies.

*Note:* After the updated Week 4 Lab Instructions (2.0) were released, the Groq model produced clearer severity classifications (MEDIUM, INFORMATIONAL, CRITICAL), which changed my earlier conclusion that the Zero-Shot model was the most accurate. Based on the updated results, the Groq LLM provided the most meaningful contextual classification. The NER model still struggled to extract entities even with the revised instructions, likely because the cybersecurity alerts mainly contain technical indicators (IP addresses, hostnames, and system identifiers) rather than traditional named entities such as people, organizations, or locations.

### Fastest / most practical
The HuggingFace sentiment and zero-shot models were the fastest and easiest to integrate through the API. 
However, while they were efficient, they lacked the deeper contextual understanding that the Groq LLM demonstrated.


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

The Named Entity Recognition (NER) model still did not extract any entities from the cybersecurity alerts even after updated instructions and new input texts.
The revised instructions also introduced new sample input texts that included named entities intended to improve NER extraction. However, in testing the workflow the NER model still struggled to reliably detect entities, even with those updated examples. Because of this, I kept the original cybersecurity alert inputs and discussed the NER behavior as a failure case in the analysis.
As a result, the NER model was not particularly useful for analyzing these types of security log messages.

# Next Steps

If more time were available, the following improvements could be explored:

- Testing additional cybersecurity-specific NER models capable of detecting IP addresses, domains, and system identifiers.
- Expanding the dataset to include more diverse alert types.
- Adding more classification labels to improve anomaly detection.
- Comparing additional LLMs for better explanation quality.
