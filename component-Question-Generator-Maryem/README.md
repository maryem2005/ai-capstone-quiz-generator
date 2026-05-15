# AI Question Generator — Maryem Elgebaly
 
**Component:** Question Generator (AI Core)  
**Project:** AI-Powered Quiz Generator  
**Team:** Maria (Integration), Ryan (Data Ingestion), Maryem (Question Generator)  
**Tools:** n8n · Flowise · Airtable · Groq (llama-3.3-70b-versatile)
 
---
 
## What It Does
 
The Question Generator is the AI core of the quiz system. It reads chunked study documents from Airtable, sends the text to a Groq LLM chain via Flowise, and automatically generates multiple choice and true/false quiz questions. Each question is written back to Airtable with full metadata including topic, difficulty, question type, correct answer, and explanation.
 
**The full pipeline this component powers:**  
Documents → Question Generation → Questions Table → Quiz Delivery → Grading → Performance Tracking
 
---
 
## My Contributions
 
- Built the n8n Question Generator workflow end-to-end
- Integrated Flowise LLM chain with Groq (llama-3.3-70b-versatile) for question generation
- Fixed metadata fields so topic, document, and generated_at populate correctly on every question
- Built the Quiz Scoring - Responses Table workflow (Step 14) — auto-grades every quiz submission within 1 minute
- Built the Quiz Grading - Performance Table workflow (Step 15) — creates performance records dynamically for every user
- Fixed True/False grading case sensitivity bug (.toLowerCase() fix)
- Fixed response_type hardcoding so true_false and mcq populate correctly
- Fixed Performance table dynamic matching so all 15 users get correct records (not just one hardcoded record)
- Implemented confidence-based routing — hard questions routed to review queue, easy/medium go forward
- Ran full end-to-end pipeline test (Step 12) confirming Documents → Questions → Responses → Performance works automatically
---
 
## How It Connects to the System
 
| Direction | Component | What passes |
|-----------|-----------|-------------|
| Input | Data Ingestion (Ryan) | Documents table records with status = ready and chunked_sections populated |
| Output | Questions Table | question_text, question_type, difficulty, correct_answer, explanation, topic, document, generated_at |
| Output | Responses Table | Auto-graded responses with is_correct, score_awarded, response_type, missed_topic |
| Output | Performance Table | Per-user performance records with score_percent, passed, weak_topics, attempt_id |
 
---
 
## Airtable Schema (My Tables)
 
**Questions Table**
- question_text, question_type, difficulty, correct_answer, explanation
- topic, document (linked to Documents), generated_at, ai_generated, generation_id, confidence_score
**Responses Table**
- submitted_answer, is_correct, score_awarded, response_type, feedback
- submitted_at, attempt_id, missed_topic, status (graded), performance (linked)
**Performance Table**
- user_id, quiz, total_questions, correct_count, incorrect_count
- score_raw, score_percent, passed, weak_topics, attempt_id
- started_at, completed_at, created_at, is_completed, responses (linked)
---
 
## Key Design Decisions
 
**1. Groq over other models**  
Tested 4 models in Week 4 (HuggingFace Sentiment, Zero-Shot, NER, and Groq Llama 3). Groq provided the strongest contextual understanding for generating meaningful quiz questions from academic text.
 
**2. Flowise for LLM chain**  
Used Flowise to build and manage the question generation chain, making it easier to adjust prompts and test outputs without touching the n8n workflow directly.
 
**3. Upsert instead of Update for Performance records**  
Replaced the Update record node with a Create or Update (upsert) node matching on user_id. This fixed the bug where only one hardcoded record got updated — now every user gets their own Performance record created dynamically.
 
**4. .toLowerCase() for grading**  
Applied case normalization to both submitted_answer and correct_answer before comparison in the IF node. This fixed True/False grading failures caused by capitalization mismatches (e.g. "false" vs "False").
 
**5. Dynamic response_type**  
Changed response_type from a hardcoded dropdown to a dynamic expression pulling question_type from the Questions table, so true_false and mcq populate correctly per question.
 
---
 
## Workflows
 
| Workflow | Purpose |
|----------|---------|
| Question Generator | Reads Documents table → generates questions via Groq → writes to Questions table |
| Quiz Scoring - Responses Table | Triggered on form submission → grades answer → updates Responses table |
| Quiz Grading - Performance Table | Triggered on graded response → creates/updates Performance record per user |
 
---
 
## Known Limitations
 
- **Short answer grading** uses exact text match, not semantic matching. A correct answer worded differently will be marked wrong.
- **Old questions (records 1–193)** were generated before the metadata fix and are missing topic/document fields. All new questions populate correctly.
- **Quizzes document field** is a single line text field manually filled in — not a linked record field.
- **Old test records** with user_id: "6" are filtered out of workflows by the Airtable Trigger formula.
---
 
## End-to-End Test Results (Step 12)
 
Traced one full record through the pipeline using Document Record 8 (World War II, World History):
 
- ✅ Questions 938–943 generated with topic, document, generated_at all populated
- ✅ Response Record 60 created for user Step12Test, auto-graded within 1 minute
- ✅ Performance Record 33 created automatically with score_percent, weak_topics, passed all correct
- ✅ Full pipeline confirmed working without manual intervention
---
 
## Week 4 — Model Comparison
 
Tested 4 models on 5 text samples to select the best model for question generation.
 
| Model | Result |
|-------|--------|
| HF Sentiment (distilbert-sst-2) | Not useful — sentiment only |
| HF Zero-Shot (bart-large-mnli) | Too generic |
| HF NER (bert-large-NER) | Entity recognition only, not generative |
| Groq Llama 3 8B | ✅ Best — strong contextual understanding |
 
**Finding:** Groq Llama 3 selected for production use.
 
---
 
## Week 5 — AutoML & Model Training
 
Trained a custom model and compared generic vs fine-tuned models.
 
| Metric | Result |
|--------|--------|
| Accuracy | 50% |
| Precision | 50% |
| Recall | 100% |
| F1 | 67% |
 
**Finding:** CoLA fine-tuned model best for evaluating question quality — detects grammatical correctness which sentiment models cannot.
 
