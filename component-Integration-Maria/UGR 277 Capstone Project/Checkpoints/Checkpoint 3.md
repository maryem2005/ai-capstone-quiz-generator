# Checkpoint 3: System Redesign & Recovery

Post-Checkpoint 2: System Breakdown

After Checkpoint 2, significant architectural weaknesses became visible.

---

## Major Problems Identified

### Overly Coupled Workflows

Ryan and Maryem’s workflows were tightly interconnected.

Consequences:
- debugging became difficult
- failures cascaded
- ownership boundaries became unclear
- small changes caused unrelated breakages

---

### Missing User Interface

Originally:

- no real PDF upload experience
- no intuitive user workflow
- no complete user interaction model

The project lacked a true front-facing user journey.

---

### Manual Workflow Dependency

Automation was incomplete.

Users required:
- manual record updates
- manual workflow execution
- manual intervention

This contradicted automation goals.

---

### Metadata Propagation Failure

Critical metadata was consistently missing:

- timestamps
- IDs
- sources
- links
- confidence values
- topic tracking

---

### Logic Errors

Observed failures:

- inverted boolean conditions
- broken field references
- undefined variables
- Flowise payload mismatches
- HTTP request failures

---

### Excessive Complexity

The architecture had become difficult to maintain.

Problems:
- redundant tables
- inconsistent naming
- unclear data flow
- excessive dependency chains

---

# Decision to Rebuild

At this point, incremental fixes were no longer practical.

The decision was made to rebuild the implementation.

The redesign preserved useful concepts while replacing unstable implementation choices.

Goal:

deliver a stable, demonstrable, functional end-to-end system.

---
The rebuild involved both database redesign and workflow restructuring.

---

# Airtable Schema Redesign

## Documents Table

Fields:

- record_id
- created_at
- status
- source
- file_upload
- course
- topic
- raw_text
- clean_text
- chunked_sections
- chunk_count
- file_name
- questions
- created_by

Purpose:
Document ingestion and preprocessing.

---

## Questions Table

Fields:

- question_text
- correct_answer
- question_type
- record_id
- status
- source
- created_at
- document
- option_a
- option_b
- option_c
- option_d
- explanation
- difficulty
- topic
- quizzes
- question_number
- responses
- confidence_score

Purpose:
AI-generated question storage.

---

## Quizzes Table

Fields:

- source
- status
- record_id
- document
- created_at
- topic
- quiz_title
- user_id
- difficulty
- question_count
- question_type
- started_at
- completed_at
- responses
- performance
- generation_id

Purpose:
Quiz session tracking.

---

## Responses Table

Fields:

- record_id
- status
- source
- created_at
- quiz
- performance
- user_id
- attempt_id
- submitted_1
- submitted_2
- submitted_3
- submitted_4
- submitted_5
- submitted_6
- submitted_7
- submitted_8
- submitted_9
- submitted_10
- is_correct
- feedback
- response_type
- submitted_at
- graded_at
- score_awarded
- missed_topic
- created_by

Purpose:
Stores full quiz submission attempts.

Design decision:
Entire quiz stored in one response record.

Benefits:
- simpler grading logic
- simpler UI
- easier workflow automation

---

## Performance Table

Fields:

- record_id
- status
- source
- created_at
- quiz
- response
- user_id
- attempt_id
- total_questions
- correct_count
- incorrect_count
- score_raw
- score_percent
- weak_topics
- overall_feedback
- started_at
- completed_at
- confidence_score

Purpose:
Performance analytics.

---

# n8n Workflow Redesign

The original architecture was replaced with three modular workflows.

---

## Workflow 1: Document Ingestion + AI Generation

Responsibilities:

- PDF upload intake
- Airtable trigger detection
- text extraction
- cleaning
- chunking
- HTTP request to Flowise
- AI question generation
- Airtable question insertion

Resolved issues:
- broken prompt variables
- undefined JSON payload references
- Flowise request failures

---

## Workflow 2: Quiz Delivery + Submission

Responsibilities:

- quiz selection
- quiz rendering
- answer submission
- response storage

Known Airtable limitation:

Airtable cannot dynamically render interactive question/answer layouts like a custom frontend.

Workaround:

Users:
1. view questions
2. write answers
3. submit answers together

---

## Workflow 3: Grading + Analytics

Responsibilities:

- answer comparison
- correctness evaluation
- scoring
- feedback generation
- weak topic detection
- performance record creation

---

# Flowise Integration Lessons

Critical discovery:

Every time a Flowise chatflow changes:

- deployment endpoint changes
- HTTP request URL must be updated in n8n

Failure to update breaks integration.

---

## Prompt Engineering Improvements

Final Flowise prompt was redesigned to improve output quality.

Results:

- low repetition
- better topic coverage
- stronger distractors
- clearer explanations
- higher educational quality

---

# Final Functional Capabilities

The final implementation supports all of the following below:

document upload  
AI preprocessing  
chunk generation  
AI question generation  
quiz creation  
user quiz participation  
response submission  
automated grading  
performance analytics  
weak topic detection  
AI-generated feedback

---

# Known Limitation

## Deployment Automation Constraint

The workflows function correctly.

However:

attempting full deployment triggered n8n execution limit restrictions.

As a result:

manual execution is required.

This is an environment/platform limitation rather than a design flaw.

---

# Validation / QA

Final testing included:

- workflow re-audit
- field mapping verification
- execution path testing
- integration validation
- schema review
- AI output quality testing

---

# Lessons Learned

This project highlighted several real-world engineering lessons:

- modular architecture matters
- tightly coupled systems are difficult to maintain
- metadata integrity is critical
- automation requires trigger reliability
- low-code tools introduce platform constraints
- debugging integrations requires full end-to-end validation
- prompt engineering significantly affects AI output quality

---

# Final Outcome

Despite major architectural setbacks, the project was successfully redesigned into a stable working prototype that demonstrates a full AI-assisted educational assessment workflow.

The final implementation is significantly more robust, modular, and maintainable than the original architecture.
