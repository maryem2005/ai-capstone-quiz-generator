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
 
This component also includes the Quiz Question Assignment workflow, which automatically links newly generated questions to the correct quiz by topic.
 
**Pipeline this component covers:**
 
```
Quizzes Table (questions auto-assigned by topic)
       ↓
Student submits via Airtable form
       ↓
Responses Table (auto-graded — Step 14)
       ↓
Performance Table (scores calculated — Step 15)
```
 
---
 
## How It Connects to the System
 
| Direction | Component | What Passes |
|-----------|-----------|-------------|
| Input | Question Generator (Maryem) | Questions table with `status = generated`, `topic`, `question_type`, `correct_answer`, `explanation` |
| Input | Data Ingestion (Ryan) | Documents table with `status = ready` |
| Output | Responses Table | `is_correct`, `score_awarded`, `response_type`, `feedback`, `missed_topic`, `submitted_at`, `graded_at`, `status: graded` |
| Output | Performance Table | `score_percent`, `passed`, `weak_topics`, `correct_count`, `incorrect_count`, `is_completed`, `completed_at` |
 
---
 
## What Was Built
 
### 1. Quizzes Table (Airtable)
 
Three quizzes were created and linked to question records:
 
| Quiz | Linked Questions | Difficulty | User |
|------|-----------------|------------|------|
| Biology Practice Quiz 1 | Records 191, 206, 192, 193 | Easy | Maryem |
| Psychology Practice Quiz 1 | Records 392, 383, 384, 382 | Medium | Maryem |
| World History Practice Quiz 1 | Records 194, 389, 195, 196 | Medium | Maryem_WH |
 
Fields manually populated for each quiz record: `created_at`, `user_id`, `document` (typed as plain text since quiz creation was never automated).
 
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
 
### Workflow 1 — Quiz Question Assignment
 
Automatically assigns newly generated questions to the correct quiz by matching on topic. Runs every minute and skips any quiz that already has questions linked.
 
**Node 1 — Schedule Trigger**  
Runs every minute automatically.
 
**Node 2 — Search Questions**  
Searches the Questions table for all records with `status = generated`.
 
**Node 3 — Code in JavaScript**  
Groups questions by topic and collects their Airtable record IDs:
 
```javascript
const items = $input.all();
const groups = {};
 
for (const item of items) {
  const topic = item.json.fields.topic;
  const status = item.json.fields.status;
 
  if (status !== "generated") continue;
  if (!groups[topic]) groups[topic] = [];
  groups[topic].push(item.json.id);
}
 
return Object.entries(groups).map(([topic, ids]) => ({
  json: {
    topic,
    questionIds: ids,
    questionIdsString: ids.join(',')
  }
}));
```
 
**Node 4 — Create Quiz**  
Creates a new Quiz record per topic group.
 
**Node 5 — Update Record**  
Links all question IDs back to the newly created Quiz record using `{{ $('Code in JavaScript').item.json.questionIds }}`. Execute Once is set to OFF so all items are processed.
 
---
 
### Workflow 2 — Quiz Delivery & Scoring — Step 14 (Per-Question Grading)
 
Handles per-question auto-grading triggered on every new form submission. Runs every minute and grades any response where `status` is blank.
 
**Node 1 — Airtable Trigger**  
Polls every minute for new ungraded responses using the formula `{status} = BLANK()`.
 
**Node 2 — Wait**  
Waits 5 seconds before proceeding. This ensures Airtable has fully saved all form fields before n8n reads them. Without this, the trigger returns only 2 fields instead of the full record.
 
**Node 3 — Get a Record**  
Fetches the full Response record from the Responses table using `{{ $json.id }}`.
 
**Node 4 — Get a Record1**  
Fetches the linked Question record from the Questions table using:
```
{{ $('Airtable Trigger').item.json.fields.question[0] }}
```
The `[0]` index is required because Airtable returns linked record fields as arrays. This node must point to the Questions table, not the Responses table.
 
**Node 5 — IF Node**  
Compares `submitted_answer` against `correct_answer` after normalizing both to lowercase:
```
{{ $('Airtable Trigger').item.json.fields.submitted_answer.toLowerCase() }}
{{ $('Get a record1').item.json.fields.correct_answer.toLowerCase() }}
```
Routes to TRUE branch if they match, FALSE branch if not.
 
**Node 6 — Update Record (TRUE branch)**  
```
is_correct        → ON
score_awarded     → 1
feedback          → {{ $('Get a record1').item.json.fields.explanation }}
response_type     → {{ $('Get a record1').item.json.fields.question_type }}
correct_answer    → {{ $('Get a record1').item.json.fields.correct_answer }}
graded_at         → {{ new Date().toISOString() }}
status            → graded
```
 
**Node 7 — Update Record1 (FALSE branch)**  
```
is_correct        → OFF
score_awarded     → 0
feedback          → {{ $('Get a record1').item.json.fields.explanation }}
response_type     → {{ $('Get a record1').item.json.fields.question_type }}
missed_topic      → {{ $('Get a record1').item.json.fields.topic }}
correct_answer    → {{ $('Get a record1').item.json.fields.correct_answer }}
graded_at         → {{ new Date().toISOString() }}
status            → graded
```
 
Note: `performance` is intentionally not mapped here — it does not exist at grading time and is written back by the Performance Calculator after Step 15 completes.
 
**Node 8 — Merge**  
Recombines TRUE and FALSE branches into a single path.
 
**Node 9 — Create or Update a Record**  
Creates an initial Performance record with the score data.
 
---
 
### Workflow 3 — Quiz Delivery & Scoring — Step 15 (Full Score Calculation)
 
A separate workflow that handles full quiz completion scoring after all per-question grading is done. Also writes the Performance record ID back to each linked Response record.
 
**Node 1 — Airtable Trigger**  
Polls every minute. Formula:
```
AND({status} = "graded", {performance} = BLANK(), {quiz} != "", {user_id} != "", {user_id} != "6")
```
Filters out old broken records with numeric user IDs and ensures only properly structured responses are processed.
 
**Node 2 — Wait**  
Waits 5 seconds to ensure all data is fully saved.
 
**Node 3 — Search Records**  
Fetches all graded responses scoped to the triggered user:
```
AND({status} = "graded", {performance} = "", {quiz} != "", {user_id} = "{{ $('Airtable Trigger').first().json.fields.user_id }}")
```
 
**Node 4 — Code in JavaScript**  
Groups responses by `user_id + quiz + attempt_id`, calculates scores, and collects Response record IDs for linking:
 
```javascript
const responses = $input.all();
const groups = {};
 
for (const r of responses) {
  const user_id = r.json.fields?.user_id || "unknown";
  const quiz = r.json.fields?.quiz?.[0] || "unknown";
  const attempt_id = r.json.fields?.attempt_id || "";
  const response_id = r.json.id;
  const key = `${user_id}__${quiz}__${attempt_id}`;
 
  if (!groups[key]) {
    groups[key] = {
      user_id,
      quiz,
      attempt_id,
      response_ids: [],
      responses: []
    };
  }
  groups[key].response_ids.push(response_id);
  groups[key].responses.push(r);
}
 
const results = [];
for (const key of Object.keys(groups)) {
  const group = groups[key];
  const groupResponses = group.responses;
  const total = groupResponses.length;
  const correct = groupResponses.filter(r =>
    r.json.fields.is_correct === true ||
    r.json.fields.is_correct === "checked"
  ).length;
  const incorrect = total - correct;
  const score_percent = Math.round((correct / total) * 100);
  const passed = score_percent >= 70;
  const weak_topics = [...new Set(
    groupResponses
      .filter(r =>
        r.json.fields.is_correct !== true &&
        r.json.fields.is_correct !== "checked"
      )
      .map(r => r.json.fields.missed_topic)
      .filter(Boolean)
  )].join(", ");
 
  results.push({
    json: {
      user_id: group.user_id,
      quiz: group.quiz,
      attempt_id: group.attempt_id,
      response_ids: group.response_ids,
      total_questions: total,
      correct_count: correct,
      incorrect_count: incorrect,
      score_percent,
      passed: passed === true ? true : false,
      weak_topics
    }
  });
}
 
return results;
```
 
**Node 5 — IF Node**  
Checks `total_questions > 0` before updating the Performance record. This prevents partial scoring from running before the student has finished.
 
**Node 6 — Create or Update a Record (Upsert)**  
Matches on `user_id + quiz`. Creates a new Performance record if none exists, or updates the existing one. No hardcoded record IDs — works for any user automatically.
 
Fields written:
```
user_id           → {{ $json.user_id }}
quiz              → {{ $json.quiz }}
total_questions   → {{ $json.total_questions }}
correct_count     → {{ $json.correct_count }}
incorrect_count   → {{ $json.incorrect_count }}
score_percent     → {{ $json.score_percent }}
passed            → {{ $json.passed }}
weak_topics       → {{ $json.weak_topics }}
attempt_id        → {{ $json.attempt_id }}
responses         → {{ $json.response_ids }}
created_at        → {{ new Date().toISOString() }}
started_at        → {{ new Date().toISOString() }}
is_completed      → true
status            → completed
completed_at      → {{ new Date().toISOString() }}
```
 
**Node 7 — Update Record (Write-back)**  
After the Performance record is created/updated, writes the Performance record ID back to each linked Response record:
```
Table:        Responses
Record ID:    {{ $('Create or update a record').item.json.fields.responses[0] }}
performance:  {{ [$('Create or update a record').item.json.id] }}
```
 
---
 
## Grading Logic
 
**Passing threshold:** 70% and above = PASSED
 
**Score calculation:**
```
score_percent = Math.round((correct_count / total_questions) * 100)
passed        = score_percent >= 70
weak_topics   = deduplicated list of topics from all incorrect responses
```
 
**Case normalization:**  
Both `submitted_answer` and `correct_answer` are converted to lowercase via `.toLowerCase()` before comparison. This fixed a bug where `false` (lowercase) would not match `False` (capital F), causing correct True/False answers to be graded as wrong.
 
**Dynamic `response_type`:**  
`response_type` pulls dynamically from the Question record's `question_type` field. This ensures `true_false` and `multiple_choice` (and any future types) populate correctly without hardcoding.
 
---
 
## Test Results
 
### Step 14 — Per-Question Grading
 
| Record | User | Submitted | Correct Answer | Result | Score |
|--------|------|-----------|---------------|--------|-------|
| 41 | Maryem | False | False | Correct | 1.0 |
| 47 | Maryem_test5 | False | False | Correct | 1.0 |
| 48 | Maryem_test6 | True | False | Wrong | 0.0 |
| 68 | holasss | false | False | Correct (after toLowerCase fix) | 1.0 |
| 72 | Mary (Photosynthesis) | — | — | Graded, performance record 47 created | — |
| 73 | panda2.0 (World History) | — | — | Graded, performance record 51 created | — |
 
Records 47 and 48 were auto-graded within 1 minute of form submission with no manual intervention.
 
### Step 15 — Full Score Calculation (World History Practice Quiz 1)
 
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
| `score_percent` | 50% |
| `passed` | False |
| `is_completed` | True |
| `completed_at` | 5/9/2026 6:47pm |
 
Confirmed working across 15 users including Maryem_WH (50%, failed), Maryem_test5 (100%, passed), and Step12Test (0%, failed, weak_topics: American History).
 
### End-to-End Test (May 18, 2026)
 
Submitted Photosynthesis notes through Ryan's ingestion form. The full pipeline ran automatically:
1. Document chunked by Ryan's workflow
2. Flowise generated Photosynthesis questions
3. Quiz Assignment workflow created Photosynthesis Practice Quiz (record 111)
4. Questions linked to quiz automatically
5. Form submission graded within 1 minute
6. Performance record created automatically
---
 
## Bugs Fixed
 
| Bug | Root Cause | Fix Applied |
|-----|-----------|-------------|
| Airtable trigger stuck on old records | Last "seen" record had no `question` field linked, causing `Get a record1` to fail with NOT_FOUND on every poll | Deleted all old response records with no question field linked |
| Airtable Automation 1 racing with n8n trigger | Automation was updating `status` immediately on form submission, so by the time n8n polled (every 1 min), `status` was no longer blank and n8n skipped the record | Turned off Airtable Automation 1 |
| Duplicate Cell Biology quiz records | Questions 322, 323, 324 were linked to both quiz record 107 and 110 simultaneously, confusing the grader | Removed duplicate links from questions, deleted record 110 |
| `Get a record1` fetching wrong table | Record ID was set to `$('Airtable Trigger').item.json.id` — the Response's own ID — but the table was set to Questions, so it was looking for a Question with the wrong ID | Changed Record ID to `{{ $('Airtable Trigger').item.json.fields.question[0] }}` |
| IF node routing everything to False branch | Second expression was `correct_answer_lookup[0].toLowerCase()` — a lookup array from the Response record — instead of the plain string from the Question record | Changed to `correct_answer.toLowerCase()` pulling from `Get a record1` |
| `response_type` always `multiple_choice` | Hardcoded dropdown value in both Update record nodes | Replaced with `{{ $('Get a record1').item.json.fields.question_type }}` |
| `feedback` and `missed_topic` not populating | Field expressions pointed to wrong source after restructuring the workflow | Updated all field expressions to pull from `$('Get a record1')` |
| Performance not linking back to Response | After removing `performance` from the grader nodes, the Response records had no performance link | Added a new Update record node at the end of the performance calculator to write the Performance ID back to each Response |
| True/False case sensitivity | `false` (lowercase) did not match `False` (capital F) in the IF node | Applied `.toLowerCase()` to both sides of the comparison |
| Performance table hardcoded to record 12 | Update record node was matching on a hardcoded record ID instead of dynamically finding the right user | Replaced Update record with Create or Update (upsert) matching on `user_id + quiz` |
| IF node type mismatch | n8n comparing a number against a string in the total_questions check | Enabled "Convert types where required" toggle |
| Duplicate quiz records from Quiz Assignment workflow | No deduplication check — Search Documents returned 22 items and Create Quiz ran for all 22 on every trigger | Rebuilt workflow to search Questions by topic instead, filter for quizzes with no questions linked |
| `questionIds` undefined in Update record | Expression was `{{ $json.questionIds }}` but data came from the Code node, not the previous node | Changed to `{{ $('Code in JavaScript').item.json.questionIds }}` |
 
---
 
## Additional Fixes (May 13, 2026)
 
### Fix 1 — True/False Grading Bug (Case Sensitivity)
 
True/False questions were being marked incorrect even when the right answer was submitted. A student typing `false` in lowercase would fail because the correct answer was stored as `False` with a capital F.
 
Fix applied in the IF node:
 
```javascript
// Top field:
{{ $('Airtable Trigger').item.json.fields.submitted_answer.toLowerCase() }}
 
// Bottom field:
{{ $('Get a record1').item.json.fields.correct_answer.toLowerCase() }}
```
 
Confirmed working on record 68 — submitted `false`, graded correct, `score_awarded: 1.0`.
 
### Fix 2 — response_type Hardcoding
 
`response_type` was hardcoded to `multiple_choice` for every response regardless of the actual question type.
 
Fix applied on both TRUE and FALSE branch Update record nodes:
 
```javascript
{{ $('Get a record1').item.json.fields.question_type }}
```
 
True/False questions now correctly populate `true_false`. MCQ questions populate `mcq`.
 
---
 
## Known Limitations
 
| Limitation | Detail |
|------------|--------|
| Short answer grading | Exact text match only. A correct answer worded differently will be marked wrong. Semantic grading was planned for Step 17. |
| MCQ grading via form | Airtable forms cannot dynamically display `option_a` through `option_d`. MCQ answer matching is scoped for Version 2. |
| Wrong topic labels on some questions | Some Psychology and Biology questions were tagged as World History during generation — a data quality issue from the question generator, not this workflow. |
| `quiz` field in Quizzes table | Single line text field filled manually. Quiz creation was never automated (original team member responsible was removed from the group). |
| Old test records | Records with `user_id: "6"` and records missing a `question` link are filtered out of all workflows via Airtable Trigger formulas. |
| Performance table cleanup | ~63 Performance records accumulated during testing, most empty or broken. Manual cleanup needed before demo. |
 
---
 
## Airtable IDs
 
| Item | ID |
|------|----|
| Base ID | `appGUdsNn18sbSOH3` |
| Quizzes Table ID | `tbl97PjXIynqTV4sY` |
 
---
 
## Workflows Summary
 
| Workflow | Trigger | Input | Output |
|----------|---------|-------|--------|
| Quiz Question Assignment | Schedule (every 1 min) | Questions table, `status = generated` | Quizzes table — quiz created and questions linked by topic |
| Quiz Delivery & Scoring — Step 14 | Airtable polling (every 1 min), `{status} = BLANK()` | New form submission in Responses table | Graded response with `is_correct`, `score_awarded`, `feedback`, `response_type`, `missed_topic`, `status: graded` |
| Quiz Delivery & Scoring — Step 15 | Airtable polling (every 1 min), graded unlinked responses | All graded responses for a user's quiz | Performance record upserted per user + performance ID written back to each Response |
