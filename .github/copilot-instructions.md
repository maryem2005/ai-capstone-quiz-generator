# Capstone Project Context

## Project
- **Name:** AI-Powered Quiz Generator

- **Team:** 
  - Ryan — Ingestion
  - Maryem — AI Core / Question Generation
  - Noeleen — Specialist / Quiz Delivery & Scoring
  - Maria — Integration / End-to-End Testing / Dashboard Planning

- **What it does:**  
This project is an AI-powered quiz generation platform that allows users to upload study materials such as text-based PDFs or notes, automatically process the content, generate quiz questions using AI, deliver quizzes to users, grade responses, and track performance. The goal is to help students study more efficiently through personalized AI-generated assessments and feedback.

- **Project type:** Educational AI Learning System / AI Quiz Generator

---

## Architecture

- **Ingestion:**  
Users upload study materials through Airtable forms. Ryan’s n8n workflow receives uploaded content, extracts text, cleans it, chunks it into sections, and writes processed content into the Documents table.

- **AI Core:**  
Maryem’s AI workflow reads processed document chunks from Airtable and uses Flowise + Groq (llama-3.3-70b-versatile) to generate quiz questions. Questions include multiple choice, true/false, and short answer formats. Generated questions are written into the Questions table with explanations, difficulty levels, correct answers, and topic metadata.

- **Specialist:**  
Noeleen’s component handles quiz delivery, response collection, grading logic, feedback generation, and performance tracking. User quiz answers are stored in Responses, graded, and linked to Performance tracking.

- **Integration:**  
Maria’s component focuses on connecting all components into a complete end-to-end system, validating workflow handoffs, identifying schema mismatches, debugging automation issues, and planning dashboard/reporting features. The dashboard is not fully finalized yet, but will likely focus on monitoring document processing, generated questions, quiz attempts, scores, weak topics, and workflow errors.

---

## Tech Stack

- n8n Cloud
- Flowise Cloud
- Groq API (llama-3.3-70b-versatile)
- Airtable
- Airtable Forms
- Airtable Interfaces
- GitHub
- GitHub Copilot
- PDF document upload processing

---

## Airtable Schema

### Documents

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | ingestion | uploaded, extracting, cleaned, chunked, ready, error |
| source | text | ingestion/manual | |
| file_upload | attachment | user | |
| title | text | user | |
| course | text | user | |
| topic | text | ingestion | |
| raw_text | long text | ingestion | |
| clean_text | long text | ingestion | |
| chunked_sections | long text | ingestion | |
| chunk_count | number | ingestion | |
| processed_text_ref | text | ingestion | |
| user_id | text | system | |
| filename | text | system | |
| questions | linked record | AI Core | |

---

### Questions

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| question_text | text | AI Core | |
| question_type | single select | AI Core | mcq, true_false, short_answer |
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | AI Core | generated |
| source | text | AI Core | ai_generated |
| document | linked record | AI Core | |
| correct_answer | text | AI Core | |
| option_a | text | AI Core | |
| option_b | text | AI Core | |
| option_c | text | AI Core | |
| option_d | text | AI Core | |
| explanation | long text | AI Core | |
| difficulty | single select | AI Core | easy, medium, hard |
| generated_at | datetime | AI Core | |
| topic | text | AI Core | |
| is_used | checkbox | system | |
| quiz | linked record | Specialist | |
| question_number | number | system | |
| ai_generated | checkbox | AI Core | |
| generation_id | text | AI Core | |
| responses | linked record | Specialist | |
| confidence_score | number | AI Core | |

---

### Quizzes

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| topic | text | system | |
| quiz_title | text | Specialist | |
| user_id | text | system | |
| difficulty | text | planned | |
| question_count | number | planned | |
| quiz_type | text | planned | |
| time_limit_minutes | number | planned | |
| assigned_at | datetime | planned | |
| completed_at | datetime | system | |
| responses | linked record | Specialist | |
| performance | linked record | Specialist | |
| generation_id | text | system | |
| is_active | checkbox | system | |

---

### Responses

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | Specialist | graded |
| source | text | system | |
| quiz | linked record | Specialist | |
| question | linked record | Specialist | |
| performance | linked record | Specialist | |
| user_id | text | user/system | |
| attempt_id | text | system | |
| submitted_answer | text | user | |
| correct_answer | lookup | system | |
| is_correct | checkbox | grading logic | |
| feedback | text | grading logic | |
| response_type | text | system | |
| submitted_at | datetime | system | |
| graded_at | datetime | grading logic | |
| score_awarded | number | grading logic | |
| missed_topic | text | grading logic | |
| correct_answer_lookup | lookup | system | |

---

### Performance

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| score_percent | number | grading logic | |
| weak_topics | text | grading logic | |
| strong_topics | text | grading logic | |
| overall_feedback | long text | grading logic | |
| started_at | datetime | system | |
| completed_at | datetime | system | |
| time_spent_minutes | number | system | |
| passed | checkbox | grading logic | |
| is_completed | checkbox | system | |

---

### Quiz Results

This is a support/reporting table rather than a core workflow table.

| Field | Type | Written By |
|------|------|------------|
| Submission ID | autonumber | system |
| Student Name | text | sample/manual |
| Quiz Date | datetime | system |
| Raw Score | number | grading logic |
| Total Questions | number | grading logic |
| Grade (%) | formula | Airtable |
| Struggle Topics | text | grading logic |
| AI Feedback | long text | AI logic |

---

### Errors

This is a support/debugging table intended for workflow/system error logging.

| Field | Type | Written By |
|------|------|------------|
| record_id | autonumber | system |
| error_message | text | system |
| error_type | text | system |
| timestamp | datetime | system |
| source_record | text / linked reference | system |

---

## Conventions

- Field names use snake_case where applicable
- Status values are lowercase
- Date/time fields end in `_at`
- Boolean fields use `is_` prefix
- Linked records connect workflow stages
- Airtable serves as the shared source of truth across all components

---

## Current State

### What's working
- Document upload through Airtable form
- PDF/text ingestion
- Text extraction
- Text cleaning
- Document chunking
- AI-generated quiz question creation
- True/False generation
- Short answer generation
- Multiple choice generation (partially reliable)
- Quiz record creation
- Response capture
- Grading logic
- Feedback generation
- Performance tracking
- End-to-end workflow testing

### What's in progress
- Full automation without manually clicking Execute Workflow in n8n
- Dashboard/reporting design
- Clarifying the role of Quiz Results vs Performance
- Clarifying ownership/use of Errors table
- Better multiple choice generation reliability
- Better quiz analytics and reporting
- Error logging implementation

### Known issues
- n8n workflow currently requires manually clicking “Execute Workflow”
- User no longer needs to manually update Airtable document status (this issue was fixed)
- System is not fully autonomous yet for normal end users
- Multiple choice questions can sometimes be malformed
- Some schema fields are placeholders and not fully populated
- Quiz Results overlaps with Performance functionality
- Errors table exists but is not actively used
- Question numbering logic may be incomplete
- Performance feedback fields are inconsistently populated
- System likely works best with text-based PDFs rather than scanned/image PDFs

### Next milestone
Checkpoint 2 — one complete record flowing end-to-end automatically across all components without manual intervention

---

## Repository Structure

```text
/project-root
  /ingestion
  /ai-core
  /specialist
  /integration
  /.github
    copilot-instructions.md
  /docs
  /screenshots
  README.md
```
