# Integration, Testing & Presentation
**Project 5: AI-Powered Quiz Generator**

## What This Component Does

The Integration component is the project backbone. While the other three components each own a single workflow, Integration owns the shared infrastructure that all of them depend on — the Airtable schema, the test data, the dashboard, and the end-to-end verification that everything connects correctly.

Nothing in this project works without what this component provides first.

---

## Responsibilities

### 1. Airtable Schema Design
Designed all 6 tables in the shared Airtable base before any teammate wrote a workflow. Every field name, field type, status value, and linked record relationship was defined and documented in `docs/data-standards.md` and agreed on as a team in Week 4.

Tables owned:
- `documents` — raw study material inputs (written by Ingestion)
- `questions` — generated quiz questions (written by Question Generator)
- `quizzes` — quiz session records (written by Quiz Delivery)
- `responses` — student answers and scores (written by Quiz Delivery)
- `performance` — rollup table for weak-topic tracking (owned by Integration)

All tables share four universal fields: `record_id`, `created_at`, `status`, and `source`.

### 2. Status Lifecycle Design
Defined and enforced the status values each component uses to hand off records to the next:

```
documents:  unprocessed → chunking → chunked → error
questions:  pending → generated → reviewed → error
quizzes:    draft → active → complete
responses:  submitted → scored → reviewed
```

Status values are lowercase throughout — consistent casing is what allows n8n filter nodes to work silently without errors.

### 3. Test Data
Populated every table with realistic records before teammates began building, so no one had to wait on a live pipeline to develop.

| Table | Records | Coverage |
|---|---|---|
| `documents` | 12 | 3 subjects: Biology, U.S. History, Python Programming |
| `questions` | 35+ | All 4 question types (MC, T/F, short answer, fill-in-the-blank), all 3 difficulty levels |
| `quizzes` | 8 | Various subject/length combos |
| `responses` | 60+ | Correct, incorrect, blank, and edge-case answers |

Edge cases covered: blank submissions, unusually long free-text answers, true/false questions (no distractors), and documents too short to generate more than 2 questions.

### 4. Dashboard Views

| View | Type | Purpose |
|---|---|---|
| Score History | Chart | Accuracy rate over time per subject |
| Weak Topic Tracker | Filtered Grid | Subjects where accuracy < 60% |
| Question Bank Browser | Gallery | All questions browsable by type and difficulty |
| Quiz Completion Rates | Chart | Completed vs abandoned quizzes |
| Question Difficulty Stats | Grouped Grid | Avg. correct rate per difficulty level |
| Component Status Board | Kanban | All records by status — used during integration testing |

### 5. Integration Testing
Ran three formal checkpoints to catch field-name mismatches and schema gaps before they became Week-12 problems.

**Checkpoint 1 — End of Week 6**
Compared Ingestion's live output field names against what Question Generator expected as input. Found mismatch: `raw_content` vs `raw_text`. Resolved before Component 2 built against it.

**Checkpoint 2 — End of Week 9**
Traced one record through the full pipeline: document ingestion → question generation → quiz delivery → scoring → performance rollup. Found: Component 3 writing `subject` instead of `subject_tag`. Fixed same day.

**Checkpoint 3 — End of Week 12**
Full pipeline test at demo scale (50+ records). Confirmed rollup fields calculated correctly. Recorded demo walkthrough video as backup.

### 6. Documentation & Portfolio
- Project-level `README.md` — system overview, how pieces connect, team roles
- `docs/setup-guide.md` — step-by-step instructions for running the project from scratch
- `docs/architecture.drawio` — full system diagram
- `docs/data-standards.md` — field naming conventions, status values, and schema decisions
- GitHub Pages portfolio page — project summary with architecture diagram and component READMEs linked

---

## Inputs & Outputs

**Inputs:** Nothing — this component creates the foundation others consume.

**Outputs:**
- Fully structured Airtable base with all tables, fields, and linked records
- 30+ test records across all tables
- Dashboard views visible to any collaborator
- Documentation and demo artifacts

---

## Key Design Decisions

**One table per entity, not one table per component.**
The most common schema mistake is giving each team member their own table. Instead, every component writes to shared entity tables (`documents`, `questions`, etc.), so Integration's dashboard can show a complete picture in one row.

**Status fields as the handoff mechanism.**
Components don't call each other — they poll for records where `status = "unprocessed"`, do their work, and advance the status. This keeps components decoupled and makes the pipeline easy to debug.

**Test data before workflows.**
Providing 30+ realistic records in Week 4 let all three other components develop in parallel without waiting on a live ingestion pipeline. Edge cases in the test data (blank fields, bad formats) surfaced bugs early.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Airtable | Schema, test data, dashboard views, linked records |
| GitHub + GitHub Pages | Repo structure, portfolio page, documentation |
| draw.io | Architecture diagram |
