# AI-Powered Quiz Generator

## Overview

AI-Powered Quiz Generator is an educational AI learning platform that transforms uploaded study materials into interactive quizzes with automated grading and personalized feedback.

Users upload lecture notes or text-based PDF study materials through an Airtable interface. The system automatically processes the document, extracts and cleans the text, generates AI-powered quiz questions, delivers the quiz through a user-facing interface, evaluates responses, and provides performance analytics.

The project was designed to prioritize usability by keeping the user experience simple while automating the backend workflow across multiple AI and automation tools.

---

# Team Roles

| Team Member | Responsibility |
|-----------|----------------|
| Ryan | Document Ingestion & Processing |
| Maryem | AI Question Generation |
| Maryem | Quiz Delivery & Grading Logic |
| Maria | Integration, Workflow Validation, End-to-End Testing |
---

# Problem Statement

Students often spend significant time manually creating study questions from notes, with little structured feedback on weak areas.

This system solves that problem by automating the quiz generation and assessment process using AI, creating a faster and more personalized study workflow.

---

# Core Features

- Upload study notes through Airtable
- Automatic PDF/text extraction
- Text cleaning and chunking for AI processing
- AI-generated multiple choice, true/false, and short answer questions
- Quiz generation and delivery through Airtable Interfaces
- Automated grading workflow
- Personalized performance feedback
- Weak topic identification
- End-to-end workflow automation
- Modular architecture for future scalability

---

# System Architecture

## 1. Ingestion Layer

Users begin by uploading study material through an Airtable form.

The ingestion workflow:
- receives uploaded files
- extracts raw text
- cleans formatting noise
- splits content into smaller AI-processable chunks
- stores processed content in Airtable

### Technologies
- Airtable Forms
- Airtable
- n8n Cloud

---

## 2. AI Question Generation Layer

Processed document chunks are sent into the AI generation workflow.

This layer:
- reads chunked document text
- sends prompts to Flowise
- uses Groq-hosted LLMs to generate questions
- creates:
  - multiple choice questions
  - true/false questions
  - short answer questions
- stores generated question metadata

### Technologies
- Flowise Cloud
- Groq API
- Llama 3.3 70B Versatile

---

## 3. Quiz Delivery & Grading Layer

Generated questions are linked to a quiz object accessible through Airtable Interfaces.

This layer:
- delivers quizzes to users
- captures submitted answers
- evaluates correctness
- generates feedback
- stores performance analytics

### Technologies
- Airtable Interfaces
- Airtable Automations
- n8n

---

## 4. Integration Layer

The integration layer connects all independent workflows into one functioning end-to-end system.

Responsibilities include:
- schema validation
- linked record consistency
- workflow debugging
- automation handoff verification
- end-to-end testing

This ensures each component communicates correctly across the pipeline.

---

# Final Airtable Schema

# Airtable Schema

## Documents Table

Stores uploaded source materials and tracks their progress through the document processing pipeline.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | ingestion | uploaded, extracting, cleaned, chunked, ready, error |
| source | text | ingestion/system | |
| file_upload | attachment | user | |
| course | text | user | |
| topic | text | user | |
| raw_text | long text | ingestion | |
| clean_text | long text | ingestion | |
| chunked_sections | long text | ingestion | |
| chunk_count | number | ingestion | |
| filename | text | system | |
| questions | linked record | AI Core | |
| created_by | text | system/user | |

---

## Questions Table

Stores all AI-generated quiz questions and their associated metadata.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | AI Core | generated, reviewed, error |
| source | text | AI Core | |
| document | linked record | AI Core | |
| question_text | long text | AI Core | |
| question_type | single select | AI Core | mcq, true_false, short_answer |
| option_a | text | AI Core | |
| option_b | text | AI Core | |
| option_c | text | AI Core | |
| option_d | text | AI Core | |
| correct_answer | text | AI Core | |
| explanation | long text | AI Core | |
| difficulty | single select | AI Core | easy, medium, hard |
| topic | text | AI Core | |
| question_number | number | system | |
| quizzes | linked record | Specialist | |
| responses | linked record | Specialist | |

---

## Quizzes Table

Groups generated questions into a quiz instance that users can complete.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | Specialist | generated, active, completed |
| source | text | system | |
| quiz_title | text | Specialist | |
| topic | text | system/AI Core | |
| difficulty | single select | system | easy, medium, hard |
| question_type | text | system | |
| question_count | number | system | |
| questions | linked record | Specialist | |
| document | linked record | system | |
| user_id | text | user/system | |
| generation_id | text | system | |
| started_at | datetime | system | |
| completed_at | datetime | system | |
| responses | linked record | Specialist | |
| performance | linked record | Specialist | |

---

## Responses Table

Stores submitted quiz answers from users and grading results.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | grading logic | submitted, graded |
| source | text | system | |
| quiz | linked record | Specialist | |
| performance | linked record | Specialist | |
| user_id | text | user/system | |
| submitted_1 | text | user | |
| submitted_2 | text | user | |
| submitted_3 | text | user | |
| submitted_4 | text | user | |
| submitted_5 | text | user | |
| submitted_6 | text | user | |
| submitted_7 | text | user | |
| submitted_8 | text | user | |
| submitted_9 | text | user | |
| submitted_10 | text | user | |
| correct | checkbox | grading logic | |
| feedback | long text | grading logic | |
| response_type | text | system | |
| submitted_at | datetime | system | |
| graded_at | datetime | grading logic | |
| score_awarded | number | grading logic | |
| missed_topic | text | grading logic | |
| created_by | text | system/user | |

---

## Performance Table

Stores summarized quiz performance metrics and AI-generated feedback.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | grading logic | completed |
| source | text | system | |
| quiz | linked record | Specialist | |
| responses | linked record | Specialist | |
| user_id | text | system | |
| attempt_id | text | system | |
| total_questions | number | system | |
| correct_count | number | grading logic | |
| incorrect_count | number | grading logic | |
| raw_score | number | grading logic | |
| score_percent | percentage | grading logic | |
| weak_topics | long text | grading logic | |
| overall_feedback | long text | AI / grading logic | |
| confidence_score | percentage | AI logic | |

---

### Design Decision
Instead of creating one Airtable record per question response, the system stores all quiz answers inside a single response record.

Benefits:
- lower Airtable record usage
- simpler grading logic
- cleaner workflow automation
- faster performance aggregation

# End-to-End Workflow

## User Journey

1. User uploads lecture notes
2. Ingestion workflow extracts document text
3. Text is cleaned and chunked
4. AI generates quiz questions
5. Questions are stored in Airtable
6. Quiz record is created
7. User accesses quiz via Airtable Interface
8. User submits responses
9. Grading workflow evaluates answers
10. Performance metrics are generated
11. Feedback is returned to the user

---

# Tech Stack

## Backend / Automation
- n8n Cloud
- Airtable Automations

## AI
- Flowise
- Groq API
- Llama 3.3 70B

## Data Layer
- Airtable Base
- Airtable Linked Records

## Frontend
- Airtable Forms
- Airtable Interfaces

## Development Tools
- GitHub
- GitHub Copilot

---

# Design Philosophy

This system was intentionally designed around simplicity for the end user.

Rather than exposing backend workflows directly, users interact only with Airtable’s front-end interface while automation and AI processing occur behind the scenes.

The architecture is modular, making it easier to debug, improve, and scale each component independently.

---
# Verified End-to-End Functionality

The AI-Powered Quiz Generator has been successfully implemented as a complete end-to-end educational AI workflow.

The following functionality has been fully validated:

## Document Processing
✅ Users can upload study materials through the Airtable interface  
✅ Text-based PDF and note ingestion works successfully  
✅ Raw text extraction functions correctly  
✅ Text cleaning and preprocessing are fully operational  
✅ Document chunking for AI processing works as intended  

## AI Quiz Generation
✅ AI-generated multiple choice questions function correctly  
✅ AI-generated true/false questions function correctly  
✅ AI-generated short answer questions function correctly  
✅ Question metadata (difficulty, explanations, correct answers, topic mapping) is generated successfully  
✅ Quiz records are automatically created and linked properly  

## Quiz Delivery & User Interaction
✅ Users can access generated quizzes through the front-end interface  
✅ Users can submit quiz responses successfully  
✅ Response data is stored correctly in Airtable  
✅ End-to-end linked record relationships function correctly across all tables  

## AI Grading & Performance Analytics
✅ Automated grading workflow functions successfully  
✅ AI-generated feedback is produced correctly  
✅ Weak topic identification works successfully  
✅ Performance metrics are calculated accurately  
✅ Final quiz performance summaries are generated successfully  

## System Integration
✅ Ingestion, AI generation, quiz delivery, grading, and performance tracking function as one complete integrated workflow  
✅ Schema relationships across Documents, Questions, Quizzes, Responses, and Performance tables are functioning correctly  
✅ End-to-end testing has validated successful record flow across all components  

---

# Current Platform Constraints

While the backend workflow is fully functional, the primary limitation is the Airtable Interface layer.

Airtable provided a practical front-end solution for rapid academic development and system integration, but it introduces usability constraints for production-level deployment, including limited interface customization, restricted user interaction flexibility, and reduced adaptability for more polished educational experiences.

This limitation does not affect backend functionality, but it impacts the overall end-user experience.

---

# Future Improvements

Potential enhancements for future iterations include:

- migrating the user interface to a more flexible and user-friendly front-end platform
- support for scanned or image-based PDFs through OCR integration
- adaptive quiz difficulty based on user performance
- quiz history tracking for repeat study sessions
- richer performance analytics and study progress insights
- improved dashboard visualization for educators or administrators
- multi-user authentication and account management
- downloadable quiz reports and study summaries
- expanded AI validation for stronger question quality assurance
---
