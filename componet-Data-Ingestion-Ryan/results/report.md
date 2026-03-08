# Model Comparison Report — Week 4
**Name:** Ryan Maca 
**Date:**  March 8, 2026
**Capstone Project:** AI-Powered Quiz Generator
**My Component:** Data-Ingestion

## Test Setup
**Input dataset:** 5 [domain] text samples covering:
Test Setup
Input dataset: 5 educational text samples covering a range of content types suitable for quiz question generation:
2 clearly factual/high-value records for quiz generation (mitochondria, Shakespeare)
1 ambiguous/opinion-based record (Python vs other languages)
2 general knowledge records (WWII, water cycle)


**Models tested:**
week-04-lab-instructions.md 2026-02-26
9 / 14
1. distilbert-base-uncased-finetuned-sst-2-english (sentiment)
2. facebook/bart-large-mnli (zero-shot classification)
3. dslim/bert-large-NER (named entity recognition)
4. Groq Llama 3 8B (LLM classification)
**Evaluation criteria:** label accuracy, confidence score, speed, ease of
integration in n8n
## Results Summary
| Record | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|-----------|-----------|-------------|------|
| 1 | [label] ([score]) | [label] | [entities found] | [classification] |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |
## Analysis
**Where models agreed:** [Which records did all models classify the same way?]






**Where models disagreed:** [Which records got different classifications? What
might explain the difference?]






**Most accurate model overall:** [Based on your 5 records, which model gave the
most sensible results?]




**Fastest/most practical:** [Which model would be easiest to use at scale?]
## Recommended Models for My Capstone Component
**Component:** [Your capstone component name]
**Primary model:** [Name] — [1 sentence: why this one for your task]
**Secondary model (if applicable):** [Name] — [1 sentence: what role it plays]
**Rejected models and why:**
- [Model name]: [1-2 sentences on why it's not the right fit]
## Failure Cases and Limitations
[Describe at least one case where a model gave a wrong or surprising result.
What does this tell you about using this model in production?]
## Next Steps
[What would you test next if you had more time? More records? Different labels?
A different model you didn't test this week?]
