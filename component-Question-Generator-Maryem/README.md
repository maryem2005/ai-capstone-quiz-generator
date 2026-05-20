# Question Generator — AI Core Component Documentation

**Component:** AI Core / Question Generator & Quiz Delivery & Scoring 
**Name:** Maryem Elgebaly  
**Project:** AI-Powered Quiz Generator  
**Stack:** n8n · Flowise · Airtable · Groq API (`llama-3.3-70b-versatile`)  

---

## What This Component Does

The Question Generator is the AI brain of the quiz system. It reads chunked study documents from Airtable, sends the text to a Groq LLM chain built in Flowise, and automatically generates multiple-choice, true/false, and short-answer quiz questions. Every generated question is written back to Airtable with full metadata — topic, difficulty, question type, correct answer, and explanation — ready for quiz delivery and grading downstream.

**Pipeline this component powers:**

```
Documents → Question Generation → Questions Table → Quiz Delivery → Grading → Performance Tracking
```

---

## How It Fits Into the System

| Direction | Component | What Passes |
|-----------|-----------|-------------|
| Input | Data Ingestion (Ryan) | Documents table records with `status = ready` and `chunked_sections` populated |
| Output | Questions Table | `question_text`, `question_type`, `difficulty`, `correct_answer`, `explanation`, `topic`, `document`, `generated_at` |
| Output | Responses Table | Auto-graded responses with `is_correct`, `score_awarded`, `response_type`, `missed_topic` |
| Output | Performance Table | Per-user records with `score_percent`, `passed`, `weak_topics`, `attempt_id` |

---

## Airtable Schema

### Questions Table

| Field | Type | Description |
|-------|------|-------------|
| `question_text` | Long text | The generated question |
| `question_type` | Single select | `mcq`, `true_false`, or `short_answer` |
| `difficulty` | Single select | `easy`, `medium`, or `hard` |
| `correct_answer` | Single line text | The correct answer string |
| `explanation` | Long text | AI-generated explanation of the correct answer |
| `topic` | Single line text | Subject topic pulled from the source document |
| `document` | Linked record | Links back to the source record in the Documents table |
| `generated_at` | Date + time | ISO timestamp of when the question was generated |
| `ai_generated` | Checkbox | Always `true` for AI-generated records |
| `generation_id` | Single line text | Batch identifier for the generation run |
| `confidence_score` | Number | Self-rated model confidence (0–1) |
| `option_a` through `option_d` | Single line text | Answer choices for MCQ questions |

---

## n8n Workflows

### Workflow 1 — Question Generator

Reads document chunks from Airtable and generates quiz questions via the Flowise chain.

**Node 1 — Schedule Trigger**  
Runs every minute, polling for new documents with `status = ready`.

**Node 2 — Airtable Search Records**  
Queries the Documents table for all records where `{status} = "ready"`. Picks up whatever Ryan's ingestion workflow has marked as ready automatically — no manual triggering required.

**Node 3 — HTTP Request (Flowise / Groq)**  
Sends the `chunked_sections` field of each document to the Flowise chain endpoint. The chain runs `llama-3.3-70b-versatile` at temperature 0.3 with a prompt that instructs the model to generate at least 10 questions — minimum 4 multiple choice, 3 true/false, and 3 short answer — each with a difficulty level, correct answer, and explanation. Returns clean structured JSON.

**Node 4 — Code Node (Parser)**  
Parses the AI JSON response. Uses regex to locate the JSON object even when the model wraps it in extra surrounding text. Maps question types to Airtable's accepted values (`mcq`, `true_false`, `short_answer`). Handles all `options_json` formats including arrays, objects, and strings. Pulls `topic` and `document` directly from the upstream Chunk Text node using `$('Chunk Text').first().json` — this was the fix for Flowise dropping metadata from its response.

**Node 5 — Airtable Upsert Record**  
Writes each generated question to the Questions table matching on `question_text` to prevent duplicates. Fields written: `question_text`, `question_type`, `correct_answer`, `option_a` through `option_d`, `explanation`, `difficulty`, `topic`, `document`, `generated_at`, `ai_generated`, `generation_id`.

---

## Flowise Chain

**Model:** `llama-3.3-70b-versatile` on Groq Cloud (temperature 0.3)  
**Nodes:** Groq Chat Model → Chat Prompt Template → LLM Chain

The chain accepts a document chunk as `{question}` and returns a structured JSON object containing at least 10 questions across the three required types, each with `question_text`, `question_type`, `difficulty`, `correct_answer`, `options_json`, and `explanation`.

**Current prompt (Human Message field):**

```
Generate at least 10 quiz questions from the study notes.
Required minimum mix:
- at least 4 multiple choice questions
- at least 3 true/false questions
- at least 3 short answer questions
Return the questions in JSON format. If the source text is short, create additional
questions by covering the same concepts from different angles.
The JSON must contain an array called "questions" with at least 10 objects.
{question}
```

**Example output from a biology chunk:**
- MC: "What is the basic unit of life?" → C) Cell
- T/F: "Prokaryotic cells have a nucleus." → False
- SA: "What is the main difference between prokaryotic and eukaryotic cells?"

---

## Build Process

### Model Selection

Three candidate models were tested against four records from Ryan's dataset — biology, history, psychology, and an intentionally invalid/empty record — to evaluate quality, speed, output format, and failure behavior.

| Model | Avg Speed | Question Quality | Output Format | Bad Data Behavior | Decision |
|-------|-----------|-----------------|---------------|-------------------|----------|
| `llama-3.3-70b-versatile` | 698ms | Excellent | Clean plain text | Hallucinated new topic | Selected |
| `llama-3.1-8b-instant` | 489ms | Good | Clean plain text | Invented Titanic question | Not selected |
| `meta-llama/llama-4-scout-17b-16e-instruct` | 948ms | Good | Bold markdown | Semi-hallucinated meta-question | Not selected |

`llama-3.3-70b-versatile` was selected for consistent question quality across all subjects, clean plain-text output that n8n can parse without stripping markdown, stable production status on Groq, and detailed explanations on every valid test.

Key finding: all three models hallucinate when given empty or invalid input. This led to the action item for Ryan to add input validation in the ingestion workflow — filtering out records with fewer than 50 characters before setting `status = ready`.

---

### n8n Workflow (Direct Groq API)

Built the initial n8n workflow connecting Airtable to the Groq API directly via an HTTP Request node. Successfully generated questions from all 21 of Ryan's ready records and wrote them to the Questions table with `question_text`, `correct_answer`, four options, `explanation`, `topic`, and `ai_generated`.

Issues found and fixed during this step:

- All 124 questions generated as MCQ because the prompt only requested "one multiple choice question." Fixed by updating the prompt to specify all three question types with minimum counts.
- Rate limit hit at 30 requests per minute. Fixed by adding a 3-second Wait node between the HTTP Request and Code nodes.
- `document_id` was not yet linked back to source Documents records. Fixed in the metadata fix step below.

---

### Flowise Chain Build

Built the Question Generator chain in Flowise as a more maintainable alternative to direct Groq API calls. Using Flowise as a layer between n8n and Groq makes it possible to adjust prompts and test outputs without modifying the n8n workflow every time. Tested on biology study material — all three question types generated correctly with the expected JSON structure. The Flowise API endpoint was shared with Ryan so his n8n HTTP Request node could call it directly.

---

### End-to-End Pipeline Test

Traced one complete record through the full pipeline using Document Record 8 (World War II, topic: World History) to confirm every stage worked without manual intervention.

**Documents:** Record 8 confirmed `status: ready` with `raw_text`, `chunked_sections`, and `topic` all populated.

**Questions:** Records 938–943 generated from document 8. `document`, `topic`, `generated_at`, `difficulty`, and `question_type` all populated correctly.

**Responses:** Record 60 created for user `Step12Test`. Auto-graded within 1 minute with `status: graded`, `submitted_at`, `response_type`, `attempt_id`, and `missed_topic` all populated.

**Performance:** Record 33 created automatically for `Step12Test` with `score_percent`, `weak_topics`, `started_at`, `completed_at`, and `is_completed` all correct.

Full pipeline confirmed working end-to-end without manual intervention.

---

### Metadata Fix — Topic, Document, and generated_at

Earlier questions (records 1–193) were generated before this fix and are missing `topic` and `document` fields. The root cause was Flowise dropping all metadata from its response — when the HTTP Request sent `topic`, `record_id`, and `title` alongside the chunk, Flowise only returned the generated questions and discarded everything else.

Fix: instead of trying to recover metadata from Flowise's response, the Code node was updated to reach back to the upstream Chunk Text node directly using `$('Chunk Text').first().json` to grab `topic` and the Airtable record ID before they were lost. `generated_at` was added as `new Date().toISOString()` in the same Code node and mapped in the Create Record node.

All questions generated after this fix (records 938 onward) populate `topic`, `document`, and `generated_at` correctly.

---

### Grading Bug Fixes

**True/False Case Sensitivity**

True/False questions were being marked incorrect even when the correct answer was submitted. A student typing `false` in lowercase would fail because the stored correct answer was `False` with a capital F.

Fix applied in the IF node of the Quiz Scoring workflow:

```javascript
{{ $('Search records').item.json.fields.submitted_answer.toLowerCase() }}
{{ $('Get a record1').item.json.fields.correct_answer.toLowerCase() }}
```

Both values are normalized to lowercase before comparison. Confirmed working — submitted `false`, graded correct, `score_awarded: 1.0`.

---

**response_type Hardcoding**

`response_type` was hardcoded to `multiple_choice` for every response regardless of the actual question type. Fixed by replacing the hardcoded value with a dynamic expression on both TRUE and FALSE branches of the IF node:

```javascript
{{ $('Get a record1').item.json.fields.question_type }}
```

True/false questions now correctly write `true_false` to the Responses table.

---

### Duplicate Prevention

Both the Questions and Quizzes creation nodes were originally set to `Create`, which caused duplicate records every time the workflow ran.

Fix: changed both nodes to `Upsert` with the following match fields:
- Questions node: match on `question_text`
- Quizzes node: match on `topic`

Running the workflows multiple times no longer creates duplicate records.

---

## Known Limitations

| Limitation | Detail |
|------------|--------|
| Short answer grading | Uses exact text match. A correct answer worded differently will be marked wrong. Semantic matching was not implemented. |
| Old questions (records 1–193) | Generated before the metadata fix. Missing `topic` and `document` fields. All questions generated after the fix populate correctly. |
| MCQ form delivery | The current quiz form does not display answer options for MCQ questions. Students must type their answer rather than selecting from choices. |
| Question count per chunk | Originally generated 3 questions per document chunk. Updated to generate a minimum of 10 questions per chunk with a required type mix. |

---

## Design Decisions

**Groq over HuggingFace**  
HuggingFace Sentiment (distilbert-sst-2), Zero-Shot (bart-large-mnli), and NER (bert-large-NER) models were tested in Week 4. These models returned sentiment labels, entity tags, and classifications — none could generate contextual quiz questions from academic text. Groq Llama 3 was the only model that understood the content and returned meaningful, structured questions with explanations.

**Flowise for the LLM chain**  
Using Flowise as a layer between n8n and Groq means prompts can be adjusted and tested without modifying the n8n workflow. The chain can be updated independently at any time.

**Upsert instead of Create for Performance records**  
Switching from Update to Upsert eliminated the fragile record-ID matching that caused the single-user bug. Matching on `user_id + quiz` is more robust and scales to any number of users without changes to the workflow.

**`.toLowerCase()` for grading**  
Applied to both sides of the comparison rather than normalizing data at input time. This is a lightweight fix that works regardless of how the model stores answers or how the student types them.

**Dynamic `response_type`**  
Pulling from the Questions table at grading time rather than setting it at question creation time means the field is always accurate even if question types change or new types are added later.

---

## Workflows Summary

| Workflow | Trigger | Input | Output |
|----------|---------|-------|--------|
| Question Generator | Schedule (every 1 min) | Documents table, `status = ready` | Questions table — 10+ questions per document |
| Quiz Scoring — Responses | Form submission | Submitted form response + Questions table lookup | Responses table — fully graded record |
| Performance Calculator | Graded response record | Responses table, grouped by user | Performance table — upserted per user/quiz |


---

# Quiz Delivery & Scoring — Component Documentation
## Overview
 
The Quiz Delivery & Scoring component handles everything after questions are generated. It presents quizzes to students via an Airtable form, automatically grades each submission within one minute, calculates full quiz scores, and writes performance records per user — all without any manual intervention.
 
**This component covers two separate automated workflows:**
 
```
Form Submission → Per-Question Grading (Step 14) → Full Score Calculation (Step 15) → Performance Record
```
 
---
 
## What Was Built
 
### 1. Quizzes Table (Airtable)
 
Three quizzes were created and linked to question records:
 
| Quiz | Linked Questions | Difficulty | User |
|------|-----------------|------------|------|
| Biology Practice Quiz 1 | Records 191, 206, 192, 193 | Easy | Maryem |
| Psychology Practice Quiz 1 | Records 392, 383, 384, 382 | Medium | Maryem |
| World History Practice Quiz 1 | Records 194, 389, 195, 196 | Medium | Maryem_WH |
 
Each quiz record includes: `topic`, `document`, `quiz_title`, `user_id`, `question_count`, `quiz_type`, `created_at`.
 
---
 
### 2. Quiz Answer Submission Form (Airtable)
 
An Airtable form connected to the Responses table with the following visible fields:
 
- `user_id` — who is submitting
- `question` — which question they are answering (displays `question_text` as the dropdown label)
- `quiz` — which quiz they are taking (displays `quiz_title`)
- `submitted_answer` — their answer
The Questions table primary field was updated to `question_text` so the form dropdown shows actual question text instead of raw record IDs.
 
**Live form URL:** `https://airtable.com/appGUdsNn18sbSOH3/pagbxOi8e1F1NYims/form`
 
---
 
## n8n Workflows
 
### Workflow 1 — Quiz Delivery & Scoring (Step 14): Per-Question Grading
 
Handles per-question auto-grading triggered on every new form submission.
 
**Node 1 — Airtable Trigger**  
Polls every minute for new ungraded responses using the formula `{status} = BLANK()`. This catches every form submission that has not yet been graded.
 
**Node 2 — Wait**  
Waits 5 seconds before proceeding. This ensures Airtable has fully saved all form fields before n8n reads them — without this, the trigger would only return 2 fields instead of the full record.
 
**Node 3 — Get a Record**  
Fetches the full Response record from the Responses table using `{{ $json.id }}`.
 
**Node 4 — Get a Record1**  
Fetches the linked Question record from the Questions table using:
```
{{ $('Airtable Trigger').item.json.fields.question[0] }}
```
The `[0]` index is required because Airtable returns linked record fields as arrays.
 
**Node 5 — IF Node**  
Compares `submitted_answer` against `correct_answer` after normalizing both to lowercase with `.toLowerCase()`. Routes to the TRUE branch if correct, FALSE branch if incorrect.
 
**Node 6 — Update Record (TRUE branch)**  
Marks the response as correct and writes all grading fields:
- `is_correct` → ON
- `score_awarded` → 1
- `feedback` → pulled from the question's `explanation` field
- `graded_at` → current timestamp
- `status` → `graded`
**Node 7 — Update Record (FALSE branch)**  
Marks the response as incorrect and writes all grading fields:
- `is_correct` → OFF
- `score_awarded` → 0
- `feedback` → pulled from the question's `explanation` field
- `missed_topic` → pulled from the question's `topic` field
- `graded_at` → current timestamp
- `status` → `graded`
**Node 8 — Merge**  
Recombines the TRUE and FALSE branches so the workflow continues as a single path.
 
**Node 9 — Create or Update a Record**  
Creates a Performance record with the initial score data after grading.
 
---
 
### Workflow 2 — Quiz Delivery & Scoring (Step 15): Full Score Calculation
 
A separate workflow that handles full quiz completion scoring after all per-question grading is done.
 
**Node 1 — Airtable Trigger**  
Polls every minute for graded World History responses from `Maryem_WH` using:
```
AND({status} = "graded", {quiz} = "World History", {user_id} = "Maryem_WH")
```
 
**Node 2 — Wait**  
Waits 5 seconds to ensure all data is fully saved before proceeding.
 
**Node 3 — Search Records**  
Fetches all graded responses for the user's quiz from the Responses table:
```
AND({user_id} = "Maryem_WH", {quiz} = "World History", {status} = "graded")
```
 
**Node 4 — Code in JavaScript**  
Calculates the full quiz score using this logic:
 
```javascript
const responses = $input.all();
const total = responses.length;
const correct = responses.filter(r =>
  r.json.fields.is_correct === true ||
  r.json.fields.is_correct === "checked"
).length;
const incorrect = total - correct;
const score_percent = Math.round((correct / total) * 100);
const passed = score_percent >= 70;
const weak_topics = [...new Set(
  responses
    .filter(r =>
      r.json.fields.is_correct !== true &&
      r.json.fields.is_correct !== "checked"
    )
    .map(r => r.json.fields.missed_topic)
    .filter(Boolean)
)].join(", ");
```
 
**Node 5 — IF Node**  
Checks whether `total_questions = 4` (all questions answered) before updating the Performance record. This prevents partial scoring from running before the student has finished.
 
**Node 6 — Update Record**  
Writes all calculated fields to the Performance record: `total_questions`, `correct_count`, `incorrect_count`, `score_raw`, `score_percent`, `passed`, `weak_topics`, `is_completed`, `status`, `completed_at`.
 
---
 
## Grading Logic
 
**Passing threshold:** 70% and above = PASSED
 
**Score calculation:**
- `score_percent = (correct_count / total_questions) * 100`
- `passed = score_percent >= 70`
- `weak_topics` = deduplicated list of topics from all incorrect responses
**Case normalization:**  
Both `submitted_answer` and `correct_answer` are converted to lowercase before comparison using `.toLowerCase()`. This was required to fix a bug where `false` (lowercase) would not match `False` (capital F), causing correct True/False answers to be marked wrong.
 
**Dynamic `response_type`:**  
`response_type` is pulled dynamically from the linked Question record's `question_type` field rather than being hardcoded. This ensures `true_false` and `multiple_choice` populate correctly per question.
 
---
 
## Test Results
 
### Per-Question Grading
 
| Record | User | Submitted | Correct Answer | Result | Score | Time |
|--------|------|-----------|---------------|--------|-------|------|
| 41 | Maryem | False | False | Correct | 1.0 | 3:00pm |
| 47 | Maryem_test5 | False | False | Correct | 1.0 | 5:09pm |
| 48 | Maryem_test6 | True | False | Wrong | 0.0 | 5:11pm |
 
Records 47 and 48 were auto-graded within 1 minute of form submission with no manual intervention.
 
### Full Score Calculation (World History Practice Quiz 1)
 
4 questions submitted as `Maryem_WH`:
 
| Record | Question | Submitted | Correct | Result |
|--------|----------|-----------|---------|--------|
| 49 | Which country invaded Poland in 1939? | A | B) Germany | Wrong |
| 50 | US entered WWII before Pearl Harbor? | False | False | Correct |
| 51 | Significance of 13th Amendment? | It abolished slavery | (short answer) | Wrong* |
| 52 | Main cause of Civil War? | B | B | Correct |
 
*Short answer grading limitation — exact text match only.
 
**Final Performance Record (Record 12):**
 
| Field | Value |
|-------|-------|
| `user_id` | Maryem_WH |
| `quiz` | World History |
| `total_questions` | 4 |
| `correct_count` | 2 |
| `incorrect_count` | 2 |
| `score_raw` | 2 |
| `score_percent` | 50% |
| `passed` | False (below 70% threshold) |
| `is_completed` | True |
| `status` | completed |
| `completed_at` | 5/9/2026 6:47pm |
 
---
 
## Problems Solved During Build
 
| Problem | Fix Applied |
|---------|-------------|
| Airtable has no native webhook on free plan | Used n8n polling trigger instead (every 1 minute) |
| Trigger was stuck on cached old records | Deleted empty shell record 43, updated formula to `{status} = BLANK()` |
| Trigger only returning 2 fields per record | Added a 5-second Wait node so Airtable fully saves before n8n reads |
| Get a Record fetching wrong table | Fixed first node to use Responses table ID, second node to use Questions table ID |
| `correct_answer` pulling wrong field | Added a dedicated second Get a Record node specifically for the Questions table |
| `question` field coming back as an array | Used `[0]` to extract the record ID from the array |
| IF node type mismatch (number vs string) | Enabled "Convert types where required" toggle in the IF node |
| `quiz` field in Airtable expects an array | Removed `quiz` field from the Update Record node to avoid type errors |
| Performance records duplicating (records 12–15) | Deleted records 13–15, kept record 12 as the single source of truth |
| Search Records returning 0 results | Fixed filter formula to explicitly match `Maryem_WH` and `World History` |
| True/False grading failing on case mismatch | Applied `.toLowerCase()` to both sides of the IF comparison |
| `response_type` hardcoded to `multiple_choice` | Changed to dynamic expression pulling `question_type` from the Questions record |
 
---
 
## Additional Fixes (May 13, 2026)
 
### Fix 1 — True/False Grading Bug (Case Sensitivity)
 
True/False questions were being marked incorrect even when the right answer was submitted. A student typing `false` in lowercase would fail because the correct answer was stored as `False` with a capital F.
 
Fix applied in the IF node of the Quiz Scoring — Responses Table workflow:
 
```javascript
{{ $('Airtable Trigger').item.json.fields.submitted_answer.toLowerCase() }}
{{ $('Get a record1').item.json.fields.correct_answer.toLowerCase() }}
```
 
Confirmed working on record 68 — submitted `false`, graded correct, `score_awarded: 1.0`.
 
### Fix 2 — response_type Hardcoding
 
`response_type` was hardcoded to `multiple_choice` for every response regardless of the actual question type.
 
Fix applied by replacing the hardcoded dropdown with a dynamic expression on both TRUE and FALSE branches:
 
```javascript
{{ $('Get a record1').item.json.fields.question_type }}
```
 
True/False questions now correctly populate `true_false` in the Responses table.
 
---
 
## Known Limitations
 
| Limitation | Detail |
|------------|--------|
| Short answer grading | Uses exact text match. A correct answer worded differently will be marked wrong. Semantic grading was planned for Step 17. |
| Step 15 filter | Initially hardcoded to `Maryem_WH` and `World History`. Generalized dynamic matching was handled in the Question Generator workflow via upsert. |
| `quiz` field in Quizzes table | Single line text field filled manually — not a linked record field, since the quiz creation workflow was never built. |
 
---
 
## Workflows Summary
 
| Workflow | Trigger | Input | Output |
|----------|---------|-------|--------|
| Quiz Delivery & Scoring — Step 14 | Airtable polling (every 1 min), `{status} = BLANK()` | New form submission in Responses table | Graded response record with `is_correct`, `score_awarded`, `feedback`, `missed_topic`, `status: graded` |
| Quiz Delivery & Scoring — Step 15 | Airtable polling (every 1 min), graded responses for specific user/quiz | All graded responses for a user's quiz | Performance record with `score_percent`, `passed`, `weak_topics`, `is_completed`, `completed_at` |
