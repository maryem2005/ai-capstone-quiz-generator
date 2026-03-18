# Question Generator

Owner: Maryem Elgebaly

## Description

This component uses AI to generate quiz questions and answers from uploaded notes.

## Status

- Design complete
- Sample data prepared
- Initial implementation
- Testing
- Integration with other components
- Documentation complete

# Week 4: Model Comparison

Tested 4 AI models on 5 text samples to evaluate their suitability for the Question Generator component of our AI-Powered Quiz Generator capstone project.

## Models Tested

- HF Sentiment (distilbert-sst-2)
- HF Zero-Shot (bart-large-mnli)
- HF NER (bert-large-NER)
- Groq Llama 3 8B

## Finding

Groq Llama 3 performed best for the Question Generator component because it provided the strongest contextual understanding and most useful interpretation of input text.

See report.md for full analysis.




# Week 5: AutoML & No-Code Model Training

Trained a custom model and compared generic vs fine-tuned models.

## Custom Model Training
- Accuracy: 50%
- Precision: 50%
- Recall: 100%
- F1: 67%

## Fine-Tuned Model Comparison
- Generic: Sentiment model (not useful)
- Fine-Tuned A: CoLA (best)
- Fine-Tuned B: Sentiment (not useful)

## Finding
CoLA model is best for evaluating question quality because it detects grammatical correctness, while sentiment models fail to provide meaningful distinctions.
