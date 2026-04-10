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
Recommended mrm8488/bert-tiny-finetuned-sms-spam-detection for the AI Quiz Generator component because its fine-tuned classification capabilities can be adapted to identify key pieces of information within lecture notes, which is essential for generating meaningful quiz questions. Although the model was originally trained for spam detection, it demonstrates the ability to distinguish between different types of text patterns, which can be leveraged to separate important concepts from less relevant content in educational material. This makes it useful as a foundation for extracting core ideas, definitions, or important statements that can be transformed into quiz questions.

In comparison to the generic sentiment model, the fine-tuned models produced more consistent and structured outputs, which are better suited for integration into an automated workflow. While the generic model often returned overly broad labels such as “positive” or “negative,” the fine-tuned model provided clearer classification signals that can be more easily mapped to quiz generation logic. Overall, the fine-tuned models showed comparable performance but offered more relevant labeling, better handling of structured text, and improved alignment with the goal of transforming lecture content into study materials.
