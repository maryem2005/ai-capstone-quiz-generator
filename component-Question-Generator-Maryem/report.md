# Model Comparison Report — Week 4

 Name: Maryem Elgebaly
Date: March 6, 2026
Capstone Project: AI-Powered Quiz Generator
My Component: Question Generator

## Test Setup

 Input dataset:
5 short text samples used to simulate educational or informational content that could be converted into quiz questions.

The dataset included:

informative statements

opinion-based text

descriptive content

potentially misleading content

These examples help evaluate how different AI models interpret text that could be transformed into quiz questions.

## Models Tested

distilbert-base-uncased-finetuned-sst-2-english (sentiment analysis)

facebook/bart-large-mnli (zero-shot classification)

dslim/bert-large-NER (named entity recognition)

Groq Llama 3 8B (LLM classification)


## Evaluation Criteria

The models were evaluated based on:

usefulness of outputs for generating quiz questions

clarity of classification

ability to extract useful entities from text

ease of integration into an automated workflow using n8n


## Results Summary
| Record                   | Sentiment      | Zero-Shot        | NER Entities         | Groq          |
| ------------------------ | -------------- | ---------------- | -------------------- | ------------- |
| iPhone camera review     | NEGATIVE (1.0) | possible anomaly | MISC entity detected | INFORMATIONAL |
| Apple order complaint    | POSITIVE (1.0) | possible anomaly | ORG entity detected  | INFORMATIONAL |
| Product quality feedback | NEGATIVE (1.0) | possible anomaly | none detected        | INFORMATIONAL |
| Hacked account statement | NEGATIVE (1.0) | possible anomaly | none detected        | SEVERE        |
| Crypto giveaway message  | NEGATIVE (1.0) | possible anomaly | none detected        | CRITICAL      |


## Analysis
Where models agreed

The models generally agreed when the text clearly expressed a strong opinion or potentially problematic situation. For example, the hacked account message was classified negatively and was also marked as severe by the Groq model.

Where models disagreed

The sentiment model labeled many examples as negative because it focuses on emotional tone rather than informational context. In contrast, the Groq model interpreted the context more deeply and classified the information based on severity or informational value.

Most accurate model overall

The Groq Llama 3 model produced the most useful outputs because it provided contextual interpretations that could help generate meaningful quiz questions.

Fastest / most practical

The Hugging Face models were fast and easy to integrate using simple HTTP requests in n8n, making them useful for lightweight text processing tasks.


## Recommended Models for Capstone Component

Component: Question Generator

Primary model:
Groq Llama 3 — best suited for understanding context and generating meaningful explanations that could be used to create quiz questions.

Secondary model:
Named Entity Recognition (dslim/bert-large-NER) — helpful for identifying important entities such as organizations or key terms that could be used as quiz question topics.

Rejected models and why

Sentiment Analysis — useful for emotional tone detection but not directly helpful for generating quiz questions.

Zero-Shot Classification — useful for labeling categories but less useful for generating educational content.

## Failure Cases and Limitations

One limitation observed was that the sentiment model labeled several neutral statements as negative because it focuses on emotional tone rather than informational content. This demonstrates that models designed for classification or reasoning perform better for educational content analysis.

## Next Steps

Future improvements could include testing models specifically designed for question generation or text summarization. Larger datasets could also be used to evaluate how well models identify important concepts that should be turned into quiz questions.




