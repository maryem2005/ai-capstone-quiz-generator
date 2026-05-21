# Checkpoint 2: Full Pipeline Trace

## Objective

Checkpoint 2 evaluated whether a document could successfully move through the full pipeline:

Documents → Questions → Quizzes → Responses → Performance

without structural failure.

---

## Documents Table Review

Test document:

**The Cold War**
- History course
- topic: World History
- chunk count: 2
- status: ready

Document ingestion appeared healthy.

Confirmed:
- raw text storage
- chunk generation
- correct preprocessing

---

## Questions Table Review

AI-generated question validation confirmed:

- proper MCQ formatting
- valid distractors
- correct answer assignment
- explanation generation
- difficulty tagging

Issues:

Missing:
- document linkage
- topic propagation
- generated timestamp
- quiz link
- response link
- confidence score

---

## Quizzes Table Review

Confirmed:
- quiz generation
- question linking
- response linking
- performance linking
- question counts

Issues:

Missing:
- source
- created_at
- user_id
- generation_id
- completed_at
- document linkage

---

## Responses Table Review

Confirmed:
- response creation
- grading logic
- feedback generation
- score assignment

Issues:

Missing:
- performance linkage
- source
- attempt_id
- response_type
- submitted timestamp
- missed topic

---

## Performance Table Review

Confirmed:
- total question count
- correct count
- incorrect count
- score generation
- completion state

Issues:

Missing:
- source
- response linkage
- attempt tracking
- weak topic analysis
- strong topic analysis
- feedback insights
- started timestamp
- pass/fail status

---

## Outcome

Checkpoint 2 confirmed the pipeline was functionally operational.

However:

metadata propagation was severely incomplete.


---
