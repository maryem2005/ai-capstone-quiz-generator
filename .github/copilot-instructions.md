# Capstone Project Context

## Project
- **Name:** AI Quiz Generator
- **Team:** Ryan (Ingestion), Maryem (AI Core/Question Generator), Noeleen (Quiz Delivery & Scoring), Maria (Integration)
- **What it does:** Students upload lecture notes, textbook chapters, or any study material. The system chunks the text, generates varied practice quiz questions (multiple choice, true/false, short answer, fill-in-the-blank) with difficulty levels and answer explanations using an AI model, stores everything in Airtable, presents quizzes to students, scores their answers, and tracks which topics students struggle with.
- **Project type:** AI-Powered Study Tool — Automated Quiz Generation & Delivery System
- **Why it matters:** Active recall through quizzing is one of the most effective study techniques. This automates quiz creation from any material, demonstrating document processing, LLM prompting, and data tracking.

## Architecture
- **Ingestion — Ryan (Component 1):** n8n workflow accepts study materials pasted via Airtable form, chunks text into sections by topic, validates input (rejects chunks < 50 chars), loads chunks into Flowise vector store for searchable retrieval, writes to Documents table. Status flow: unprocessed → chunked → ready → loaded → error. Step 4 COMPLETE ✅
- **Question Generator — Maryem (Component 2):** Flowise chain reads document chunks and generates varied question types (MC, T/F, short answer, fill-in-the-blank) with difficulty levels and answer explanations. n8n workflow sends chunks to Groq API (llama-3.3-70b-versatile) via HTTP Request node, parses plain text responses, writes generated questions to Questions table in Airtable. Step 5 COMPLETE ✅. Step 7 in progress (due May 1). Step 10 upcoming (Flowise chain + confidence scoring).
- **Quiz Delivery & Scoring — Noeleen (Component 3):** Airtable form presents questions to students and accepts answers. n8n workflow scores responses, sets is_correct, provides detailed feedback referencing source material, writes results to Responses table and updates Quizzes table status.
- **Integration, Testing & Presentation — Maria (Component 4):** Designs and owns shared Airtable base (all 4 tables with linked records and rollup fields). Creates 30+ test questions across 3 subjects. Builds Airtable dashboard (score history charts, weak-topic views, quiz completion rates, question difficulty stats). Connects all components end-to-end, runs all 3 checkpoints, writes README and architecture diagram, leads demo and GitHub Pages portfolio.

## How Components Connect
Document Ingestion (Ryan) → chunks into Flowise vector store → Question Generator (Maryem) pulls chunks, creates questions stored in Airtable → Quiz Delivery (Noeleen) presents quizzes and scores answers, writes results back to Airtable → Integration (Maria) monitors all data flows via dashboard and runs checkpoints.

## Tech Stack
- n8n Cloud (workflow automation — all 4 components)
- Flowise Cloud (LLM chains — Question Generator chain, vector store)
- Groq API (LLM inference — llama-3.3-70b-versatile, selected after Step 5 model testing)
- Airtable (shared database — 4 tables, base owned by Maria)
- GitHub (group repo: MarialsCoding)
- Hoppscotch (API testing)
- draw.io (architecture diagram, owned by Maria)
- GitHub Pages (portfolio page, owned by Maria)

## Airtable Schema

### Documents (Table 1) — owned by Ryan (Ingestion)
| Field | Type | Written By | Notes |
|-------|------|-----------|-------|
| record_id | Autonumber | Auto | Primary key |
| created_at | Date+time | Auto | When submitted |
| status | Single select | All | unprocessed → chunked → loaded → error |
| title | Single line text | Ryan | e.g. "Week 3 Bio Lecture" |
| subject | Single select | Ryan | Biology, History, Math, Psychology, etc. |
| raw_text | Long text | Ryan | The pasted study material chunk |
| chunk_count | Number | Ryan | How many chunks were split from original |
| ingested_at | Date+time | Ryan | When ingestion pipeline ran |

### Questions (Table 2) — owned by Maryem (Question Generator)
| Field | Type | Written By | Notes |
|-------|------|-----------|-------|
| record_id | Autonumber | Auto | Primary key |
| created_at | Date+time | Auto | When generated |
| document_id | Link → Documents | Maryem | Which document this came from |
| question_text | Long text | Maryem | The actual question |
| question_type | Single select | Maryem | multiple_choice, true_false, short_answer, fill_blank |
| difficulty | Single select | Maryem | easy, medium, hard |
| correct_answer | Long text | Maryem | The correct answer |
| options_json | Long text | Maryem | JSON array of MC choices (if applicable) |
| explanation | Long text | Maryem | Why the answer is correct |
| subject | Single select | Maryem | Copied from the linked document |

### Responses (Table 3) — owned by Noeleen (Quiz Delivery & Scoring)
| Field | Type | Written By | Notes |
|-------|------|-----------|-------|
| record_id | Autonumber | Auto | Primary key |
| created_at | Date+time | Auto | When submitted |
| question_id | Link → Questions | Noeleen | Which question was answered |
| student_name | Single line text | Noeleen | From form input |
| student_answer | Long text | Noeleen | What student typed/selected |
| is_correct | Checkbox | Noeleen | Set by n8n scoring workflow |
| feedback | Long text | Noeleen | Explanation shown to student |
| quiz_session_id | Link → Quizzes | Noeleen | Groups responses into one quiz attempt |

### Quizzes (Table 4) — owned by Noeleen + Maria
| Field | Type | Written By | Notes |
|-------|------|-----------|-------|
| record_id | Autonumber | Auto | Primary key |
| student_name | Single line text | Noeleen | Who took this quiz |
| subject | Single select | Noeleen | Topic of the quiz |
| score | Number (formula) | Auto | % correct — calculated from Responses |
| completed_at | Date+time | Noeleen | When quiz was finished |
| status | Single select | Noeleen | in_progress, completed, scored |

## Conventions
- Field names: snake_case
- Status values: lowercase
- Date fields end in _at (created_at, ingested_at, completed_at)
- Boolean fields use is_ prefix (is_correct)
- Documents status flow: unprocessed → chunked → loaded → error
- Quizzes status flow: in_progress → completed → scored
- AI model: always use llama-3.3-70b-versatile on Groq
- Never use llama-4-scout — preview only, uses bold markdown formatting incompatible with our n8n parsing
- Output format from AI: plain text only — no markdown formatting ever
- Input validation: reject/error any chunk with fewer than 50 characters before sending to AI
- Never forward Documents with status = 'error' to Question Generator

## Current State (as of April 24, 2026)

### Completed ✅
- Step 1 (Maria): Airtable schema finalized — all 4 tables, fields, linked records, naming conventions
- Step 2 (Ryan): Test dataset created — 25 records covering normal, edge cases, bad data
- Step 3 (Maria): 30+ test records populated, Airtable views built
- Step 4 (Ryan): n8n ingestion workflow — reads Airtable, validates input, chunks text, marks ready/error. Records 1–21 = ready, Records 22–25 = error ✅
- Step 5 (Maryem): 3 candidate models tested on Ryan's records. llama-3.3-70b-versatile selected. Produces correct answers + detailed explanations in clean plain text. Avg speed 698ms.

### In Progress 🔄
- Step 6 (Noeleen, due April 29): Component README + research Flowise/Groq docs
- Step 7 (Maryem, due May 1): Set up n8n HTTP Request node → Groq API → parse response → write to Questions table in Airtable
- Step 8 (Noeleen, due May 3): Paper prototype of quiz delivery output schema
- Step 9 (Ryan): Live demo to Maria — waiting on team Zoom meeting

### Upcoming 📅
- Step 10 (Maryem, week of May 4): Build Flowise Question Generator chain with confidence scoring via prompt engineering
- Step 11 (Noeleen, week of May 4): Quiz delivery skeleton — form presents questions, accepts answers
- Step 12 (Maryem, week of May 4): Load vector store, test retrieval, refine prompts, confirm output schema writing correctly to Airtable
- Step 13 (Maria): Checkpoint 1 — compare Ryan's output vs Maryem's expected input, fix field mismatches
- Step 17 (Maryem): Confidence-based routing — questions above threshold go forward, below go to review queue

### Known Issues ⚠️
- All 3 tested models hallucinate on bad/invalid data — Ryan's IF node (Step 4) guards against this
- No model returns a confidence score natively — must implement via prompt engineering in Step 10 (Maryem)
- Records with status = 'error' must NEVER be sent to Question Generator
- Model llama-4-scout uses bold markdown — incompatible, do not use

### Next Milestone
**Checkpoint 2 (Week 9):** One complete record flows end-to-end through all 4 components without manual intervention: Ryan ingestion → Maryem question generation → Noeleen quiz delivery → Maria dashboard entry

## Repository Structure
```
MarialsCoding (GitHub repo — 189 commits)
├── component-Integration-Maria/
├── component-Question-Generator-Maryem/
├── component-quiz-delivery-Noeleen/
├── componet-Data-Ingestion-Ryan/    ← note: typo in folder name, keep consistent
├── .github/
│   └── copilot-instructions.md
├── docs/
│   └── checkpoint2-audit.md
├── screenshots/
└── prompt-log-maryem.md
```

## Test Data Summary (Ryan's Airtable — 25 records)
- Records 1–4: Biology normal (easy → hard): Cell Structure, Mitosis, Ecosystems, Genetics
- Records 5–8: History normal (easy → hard): Civil War, Battle of Gettysburg, WWII, Cold War
- Records 9–12: Math normal (easy → hard): Algebra, Fractions, Pythagorean theorem, Statistics
- Records 13–17: Edge cases (very short "Cells need energy", ambiguous notes, bullet format, very long Renaissance chunk, mixed subjects)
- Records 18–20: Bad data (empty chunk_text, missing subject, gibberish @@##)
- Records 21–25: Additional (Nervous System, Industrial Revolution, Linear Functions, Formula sheet, first-person rough notes)
- **Records 1–21: status = ready ✅**
- **Records 22–25: status = error ✅ (intentional bad data)**
