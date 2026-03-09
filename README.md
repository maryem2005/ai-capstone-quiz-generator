# Week 4: Model Comparison

Tested 4 AI models on 5 cybersecurity alert text samples to evaluate their suitability for the **AI quiz generation and classification component** of our **AI Capstone Quiz Generator** project.

## Models Tested

* HF Sentiment (distilbert-sst-2)
* HF Zero-Shot (bart-large-mnli)
* HF NER (bert-large-NER)
* Groq Llama 3 8B

## Finding

Recommended **Groq Llama 3 8B** for the **classification component** because it produced the most meaningful contextual explanations for security alerts compared to the other models, which either only produced sentiment labels or limited entity extraction.

See `report.md` for full analysis.
