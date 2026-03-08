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
1. distilbert-base-uncased-finetuned-sst-2-english (sentiment)
2. facebook/bart-large-mnli (zero-shot classification)
3. dslim/bert-large-NER (named entity recognition)
4. Groq Llama 3 8B (LLM classification)
**Evaluation criteria:** label accuracy, confidence score, speed, ease of
integration in n8n


## Results Summary
| Record | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|-----------|-----------|-------------|------|
| 1 | [The mitochondria is the powerhouse of the cell] ([POSITIVE (0.9992)]) | [possible anomaly] | [{} (none detected)] | [HIGHLY SUITABLE] |
| 2 | [World War II ended in 1945 with the surrender of Germany and Japan] ([NEGATIVE (0.9696)]) | [possible anomaly] | [MISC entities detected] | [SUITABLE] |
| 3 | [Python is a high-level programming language] ([POSITIVE (0.9960)]) | [possible anomaly] | [{} (none detected)] | [SUITABLE] |
| 4 | [The water cycle involves evaporation, condensation, and precipitation] ([NEGATIVE (0.8300)]) | [possible anomaly] | [MISC entities detected] | [SUITABLE] |
| 5 | [Shakespeare wrote Hamlet, Macbeth, and Romeo and Juliet] ([POSITIVE (0.9574)]) | [possible anomaly] | [MISC entities detected] | [HIGHLY SUITABLE] |






## Analysis
**Where models agreed:** [Which records did all models classify the same way?]
The Groq model and sentiment model broadly agreed on the highest quality records. The mitochondria and Shakespeare records were both rated positively by sentiment and classified as HIGHLY SUITABLE by Groq, reflecting strong agreement that these are the best candidates for quiz question generation. All models processed all 5 records without errors, showing consistent reliability across the dataset.



**Where models disagreed:** [Which records got different classifications? What
might explain the difference?]

The most notable disagreement was on the WWII and water cycle records, where the sentiment model returned NEGATIVE scores despite both being factually accurate and educationally valuable. The sentiment model flagged war-related vocabulary like "surrender" as negative language, while Groq correctly identified both as SUITABLE for quiz generation. The Zero-Shot model assigned "possible anomaly" to all 5 records, producing no meaningful differentiation between records across the dataset.




**Most accurate model overall:** [Based on your 5 records, which model gave the
most sensible results?]

The Groq Llama 3.1 8B model produced the most useful and contextually aware outputs. It correctly identified the mitochondria and Shakespeare records as HIGHLY SUITABLE and appropriately rated the others as SUITABLE, providing reasoning that directly supports quiz question generation decisions.


**Fastest/most practical:** [Which model would be easiest to use at scale?]


The Hugging Face models (sentiment and NER) were the fastest to integrate using simple HTTP POST requests in n8n. They return structured JSON instantly with no prompt engineering required, making them practical for lightweight preprocessing at scale.




## Recommended Models for My Capstone Component

**Component:** [Data Ingestion]


**Component:** [Data Ingestion]
**Primary model:** [ Groq Llama] — [1 sentence: why this one for your task]
**Secondary model (if applicable):** [slim/bert-large-NER] — [1 sentence: what role it plays]

Primary model: Groq Llama 3.1 8B — best suited for understanding educational context and generating meaningful classifications with reasoning that can directly inform which content should be turned into quiz questions.
Secondary model: dslim/bert-large-NER — helpful for identifying key named entities such as people (Shakespeare), locations (Germany, Japan), and technical terms that can serve as the basis for quiz question topics and answer choices.
Rejected models and why:

Sentiment Analysis (distilbert-sst-2): Misclassified the WWII and water cycle records as NEGATIVE despite being highly suitable educational content. Sentiment models are trained on consumer reviews and social media data, not academic text, making them unreliable for evaluating quiz suitability.
Zero-Shot Classification (bart-large-mnli): Assigned "possible anomaly" to all 5 records regardless of content, producing no meaningful differentiation. The model requires very carefully tuned candidate labels to be useful for educational content, and even then lacks the contextual depth of the Groq LLM.

**Rejected models and why:**
[Sentiment Analysis (distilbert-sst-2)]:
Misclassified the WWII and water cycle records as NEGATIVE despite being highly suitable educational content. Sentiment models are trained on consumer reviews and social media data, not academic text, making them unreliable for evaluating quiz suitability.
Zero-Shot Classification (bart-large-mnli): Assigned "possible anomaly" to all 5 records regardless of content, producing no meaningful differentiation. The model requires very carefully tuned candidate labels to be useful for educational content, and even then lacks the contextual depth of the Groq LLM.

## Failure Cases and Limitations
[Describe at least one case where a model gave a wrong or surprising result.
What does this tell you about using this model in production?]
The most notable failure occurred with the sentiment model on the WWII record, which received a NEGATIVE score of 0.9696 despite being factually rich and highly suitable for quiz generation. The model reacted to war-related vocabulary such as "surrender" rather than evaluating the informational quality of the text. This is a significant limitation — a filtering system based on sentiment alone would incorrectly reject valuable historical content.
Additionally, the NER model returned empty results for the mitochondria and water cycle records, failing to detect scientific terms like "mitochondria," "evaporation," and "precipitation" as named entities. This limits its usefulness for science-domain quiz generation without a domain-adapted model.

## Next Steps
[What would you test next if you had more time? More records? Different labels?
A different model you didn't test this week?]

Future testing would include models specifically fine-tuned for question generation, such as the t5-base-e2e-qg model on Hugging Face, which generates questions directly from input text. Testing with a larger dataset of 20 or more records across multiple subject domains (science, history, literature, mathematics) would also help evaluate consistency. Additionally, updating the Zero-Shot candidate labels to better match educational content such as "factual content," "opinion based," and "requires verification" and re-running the comparison would likely produce much more useful differentiation between records.
