## Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

## Name: Maryem Elgebaly
# Date: 3/18/2026
# Capstone Project: Quiz Generator
# My Component: Model comparison / question quality classification 

## Part A: Teachable Machine Training
Training Setup

Task: Ready vs Needs Review question classification

Training images per class: 20+ (assumed)

Test images per class: 5

Total training time: ~30–60 seconds

## 📝 Model A & B results  
Input: Q1. What is the capital of France? A. Berlin B. Madrid C. Paris D. Rome (ready)
Model: CoLA
Label: LABEL_1
Score: 0.977

Input: Q1. What is the capital of France? A. Berlin B. Madrid C. Paris D. Rome (ready)
Model: Sentiment
Label: neutral
Score: 0.931

Input: Q2. Which planet is known as the Red Planet? A. Earth B. Mars C. Jupiter D. Venus (ready)
Model: CoLA
Label: LABEL_1
Score: 0.972

Input: Q2. Which planet is known as the Red Planet? A. Earth B. Mars C. Jupiter D. Venus (ready)
Model: Sentiment
Label: neutral
Score: 0.930

Input: Q3. which planet is red one idk A earth B mars C sun D star (needs review) 
Model: CoLA
Label: LABEL_0
Score: 0.840

Input: Q3. which planet is red one idk A earth B mars C sun D star (needs review) 
Model: Sentiment
Label: Neutral 
Score: 0.855

Input: Q4. Who wrote romeo and juliet maybe A shakespeare B shakespeare C someone D idk (needs review)
Model: CoLA
Label: LABEL_1
Score: 0.620

Input: Q4. Who wrote romeo and juliet maybe A shakespeare B shakespeare C someone D idk (needs review)
Model: Sentiment
Label: Neutral
Score: 0.926

Input: Q5. What is 5 times 6 but think carefully??? A 11 B 30 C 60 D idk (needs review)
Model: CoLA
Label: LABEL_1
Score: 0.636

Input: Q5. What is 5 times 6 but think carefully??? A 11 B 30 C 60 D idk (needs review)
Model: Sentiment
Label: Neutral
Score: 0.873


## 📊Confusion Matrix
         |                 | Predicted: Ready | Predicted: Needs Review |
| ------------------------ | ---------------- | ----------------------- |
| **Actual: Ready**        | TP = 5           | FN = 0                  |
| **Actual: Needs Review** | FP = 5           | TN = 0                  |


## 📈My metrics 
Accuracy
= (TP + TN) / total
 = (5 + 0) / 10 = 0.5 → 50%

Precision (for Ready)
= TP / (TP + FP)
 = 5 / (5 + 5) = 0.5 → 50%

Recall
= TP / (TP + FN)
 = 5 / (5 + 0) = 1.0 → 100%

F1 Score
= 2 × (Precision × Recall) / (Precision + Recall)
 = 2 × (0.5 × 1) / (1.5) = 0.67 → 67%


## Interpretation 
The model achieved perfect recall for the Ready class, meaning it correctly identified all high-quality questions. However, it incorrectly labeled all Needs Review questions as Ready, resulting in a high number of false positives. This indicates the model is biased toward predicting the Ready class and cannot effectively distinguish poor-quality questions. To improve performance, more diverse and balanced training data is needed.



## Part B: Generic vs Fine-Tuned Model Comparison
Models Tested

Generic: distilbert-base-uncased-finetuned-sst-2-english (sentiment)

Fine-Tuned A: CoLA model — evaluates grammatical correctness

Fine-Tuned B: Sentiment model — evaluates emotional tone

## Results (based on your Airtable)
| Input | Generic       | Fine-Tuned A  | Fine-Tuned B  | Best Model |
| ----- | ------------- | ------------- | ------------- | ---------- |
| Q1    | neutral (0.9) | LABEL_1 (0.7) | neutral (0.9) | A          |
| Q2    | neutral (0.9) | LABEL_0 (0.5) | neutral (0.9) | A          |
| Q3    | neutral (0.9) | LABEL_0 (0.8) | neutral (0.9) | A          |
| Q4    | neutral (0.9) | LABEL_1 (1.0) | neutral (0.9) | A          |
| Q5    | neutral (0.9) | LABEL_1 (1.0) | neutral (0.9) | A          |


## Analysis

Generic model strengths:
The generic sentiment model produced consistent outputs and high confidence scores.

Generic model weaknesses:
It labeled all inputs as neutral, providing no meaningful distinction between good and bad questions.

Fine-tuned model advantage:
The CoLA model was able to differentiate between structured and poorly written questions, making it more useful for evaluating quality.

Biggest surprise:
The sentiment model completely failed to provide useful variation and was irrelevant for this task.


## Recommended Model for My Capstone Component

## Component: Question quality filtering

## Primary model: CoLA (Fine-Tuned A)

It actually evaluates grammar/quality

Gives meaningful variation

## Confidence threshold: 0.8

High-confidence predictions can be auto-accepted

Lower ones go to review

## Priority metric: Precision

It’s more important to avoid letting bad questions pass

False positives (bad labeled as good) are worse



## Limitations & Next Steps

The model performance is limited due to a small dataset and lack of diversity in examples. The Teachable Machine model overfit to one class, while the sentiment model was not appropriate for the task. In the future, I would collect more balanced data, test additional fine-tuned models, and potentially fine-tune my own model specifically for question quality classification.




