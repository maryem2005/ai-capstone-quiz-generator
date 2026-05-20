# Data Ingestion — Component Documentation

**Component:** Data Ingestion  
**Name:** Ryan Maca  
**Project:** AI-Powered Quiz Generator  
**Stack:** n8n · Airtable · Flowise · Groq API  

---

## What This Component Does

The Data Ingestion component is the entry point of the quiz system. It takes raw study materials submitted by students, validates and cleans the text, splits it into meaningful chunks, and writes structured data back to Airtable so the AI Question Generator can pick it up automatically.

**Pipeline this component powers:**

```
Form Submission → Status Auto-Set → Validation → Text Cleaning → Chunking → Flowise → Questions Table
```

---

## How It Fits Into the System

| Direction | Component | What Passes |
|-----------|-----------|-------------|
| Input | Student (via Airtable Form) | Title, course, topic, raw text or PDF upload |
| Output | AI Core (Maryem) | Documents table records with `status = chunked` and `chunked_sections` populated |
| Output | Errors Table | Failed records with `error_message`, `error_type`, `timestamp`, `source_record` |

---

## Airtable Schema

### Documents Table

| Field | Type | Description |
|-------|------|-------------|
| `title` | Single line text | Title of the study material |
| `course` | Single line text | Course the material belongs to |
| `topic` | Single line text | Subject topic |
| `raw_text` | Long text | Raw text submitted by the student |
| `clean_text` | Long text | Normalized text after cleaning |
| `chunked_sections` | Long text | Final chunked output sent to Maryem's AI chain |
| `chunk_count` | Number | Number of chunks produced |
| `status` | Single select | `uploaded`, `cleaned`, `chunked`, `error` |
| `file_upload` | Attachment | Optional PDF upload |
| `source` | Single line text | `manual` or `ai_generated` |

### Errors Table

| Field | Type | Description |
|-------|------|-------------|
| `record_id` | Autonumber | Auto-generated ID |
| `error_message` | Long text | The error that occurred |
| `error_type` | Single line text | Category of error |
| `timestamp` | Date + time | When the error was logged |
| `source_record` | Single line text | Title of the document that failed |

---

## n8n Workflow — Document Ingestion Pipeline

### Node 1 — When Clicking Execute Workflow (Manual Trigger)
Manually triggered to run the ingestion pipeline on all documents with `status = uploaded` or `status = cleaned`.

### Node 2 — Search Records (Airtable)
Queries the Documents table using the filter formula:
```
OR({status} = "uploaded", {status} = "cleaned")
```
Returns all records that are ready to be processed. Picks up whatever the Airtable Automation has marked as `uploaded` automatically — no manual status setting required.

### Node 3 — If3 Node (Input Validation)
Routes records to the error path if any of the following conditions are true:

| Condition | Expression | Operator |
|-----------|-----------|----------|
| Content is missing | `{{ $json.fields.raw_text \|\| $json.fields.chunked_sections }}` | is empty |
| Course is missing | `{{ $json.fields.course }}` | is empty |
| Text is too short | `{{ ($json.fields.raw_text \|\| $json.fields.chunked_sections \|\| "").length }}` | is less than 50 |

Records that fail any condition are routed to the **true branch** (error path). Records that pass all conditions continue to the **false branch** (processing path).

### Node 4 — Update Record (Error Path)
On the true branch — writes `status = error` to the Documents table for the failed record.

### Node 5 — Clean Text (Code Node)
On the false branch — strips extra whitespace, normalizes formatting, and prepares the raw text for chunking.

### Node 6 — Update Record1
Sets `status = cleaned` and writes the normalized text to the `clean_text` field.

### Node 7 — Chunk Text (Code Node)
Splits the cleaned text into meaningful sections and calculates the total chunk count.

### Node 8 — If2 Node (Chunk Validation)
Checks whether the chunking step produced any output. Routes to the error path if zero chunks were produced.

### Node 9 — HTTP Request (Flowise)
Sends the `chunked_sections` content to Maryem's Flowise chain endpoint via POST request. The chunked text is passed as the input for AI question generation.

### Node 10 — If1 Node (API Response Validation)
Checks whether the Flowise HTTP Request returned an error (`$json.error`). Routes to the error logging path on failure, continues to question writing on success.

**Continue on Fail** is enabled on the HTTP Request node so the workflow does not crash on API failures — it catches them and routes them to the error path instead.

### Node 11 — Code in JavaScript
Parses the Flowise response and prepares the question records for writing to Airtable.

### Node 12 — Create a Record (Questions Table)
Writes generated questions to the Questions table.

### Node 13 — Update Record2
Updates the Documents table record with the final `chunked_sections` and `chunk_count` values. Sets `status = chunked`.

### Node 14 — Update Record3
Final status update — confirms the record completed the full pipeline. Sets `status = chunked`.

### Node 15 — Create a Record (Errors Table — failure path)
On the If1 true branch — logs the API failure to the Errors table with `error_message`, `error_type`, `timestamp`, and `source_record`.

### Node 16 — Update Record4
On the error path — sets `status = error` on the Documents table record so it is not reprocessed.

---

## Airtable Automation — Auto-Set Status on Form Submission

**Trigger:** When a form is submitted (Submit Lecture Notes form)  
**Table:** Documents  
**Action:** Update record — sets `status = uploaded`

When a student submits the "Submit Lecture Notes" form, this automation immediately sets the new record's status to `uploaded`. This triggers the n8n workflow to pick it up automatically on its next run — no manual status changes required.

**Run history:** Confirmed running successfully. New form submissions automatically show `status = uploaded` in the Documents table.

---

## Status Values

| Status | Meaning |
|--------|---------|
| `uploaded` | New form submission — ready for n8n to pick up |
| `cleaned` | Text has been cleaned and normalized |
| `chunked` | Successfully processed end-to-end |
| `error` | Failed validation or API call — see Errors table |

---

## Steps Completed

### Step 2 — Built the Test Dataset
Created 20+ hand-built records in the Airtable Documents table covering normal study materials across Biology, History, Psychology, and Math, plus intentional edge cases:
- Missing content (no raw_text)
- Unlabeled notes (no course or topic)
- Corrupted file record
- Duplicate entries
- Records with very short or placeholder text

These records gave the rest of the team real data to develop and test against before the pipeline was complete.

### Step 4 — Built the n8n Document Ingestion Pipeline
Built the core n8n workflow end-to-end:
- Airtable form → Search records → If3 validation → Clean Text → Chunk Text → HTTP Request to Flowise → write results back to Airtable

Gave Maria a live demo of the workflow writing a real record to Airtable (Step 9 of the build sequence). Pipeline confirmed functional.

### Step 16 — Added Error Handling
- Created Errors table in Airtable with fields: `record_id` (Autonumber), `error_message`, `error_type`, `timestamp`, `source_record`
- Added If1 node to detect `$json.error` from the Flowise HTTP Request
- Enabled Continue on Fail on the HTTP Request node
- Connected the error path to a Create a Record node writing failures to the Errors table
- Connected the success path to the existing Code in JavaScript → question writing flow

### Step 18 — Added Additional Error Handling and Edge Case Coverage
Extended the If3 node with three conditions to catch ingestion-specific edge cases:

| Check | Purpose |
|-------|---------|
| `raw_text \|\| chunked_sections` is empty | Catches records with no content at all |
| `course` is empty | Catches records missing required metadata |
| Text length < 50 characters | Catches records too short to chunk meaningfully |

Using `||` on the first condition ensures records that already have `chunked_sections` populated (pre-chunked) are not incorrectly flagged as missing content.

### Status Bug Fix
Identified and fixed a bug where Update record3 was setting `status = ready` instead of `status = chunked` at the end of the pipeline. This was causing records to be reprocessed every time the workflow ran, generating duplicate questions.

**Before:** `status → ready` (picked up again on next run)  
**After:** `status → chunked` (excluded from future runs)

### Airtable Automation Build
Built a new Airtable Automation to remove the manual step of setting status after form submissions. Previously a team member had to manually change each new record's status to `uploaded` before the n8n workflow could pick it up.

**Trigger:** When a form is submitted → Submit Lecture Notes  
**Action:** Update record → status = `uploaded`

Confirmed working — new form submissions automatically show `uploaded` without any manual intervention.

---

## Data Handoff to Maryem

My component writes `chunked_sections` to the Documents table after the full pipeline completes. Maryem's Flowise chain reads from this field to generate quiz questions.

**Official handoff field:** `Documents.chunked_sections → Maryem's AI input`

Field alignment verified at Checkpoint 1 — no mismatches found between my output schema and Maryem's expected input.

---

## Problems Solved During Build

| Problem | Fix Applied |
|---------|-------------|
| If3 routing records to error even when `chunked_sections` was populated | Changed condition from `raw_text is empty` to `raw_text \|\| chunked_sections is empty` |
| Third If3 condition throwing type error (string vs number) | Changed expression to `(raw_text \|\| chunked_sections \|\| "").length` to return a number |
| Records being reprocessed on every workflow run | Fixed Update record3 from `status = ready` to `status = chunked` |
| New form submissions required manual status update | Built Airtable Automation to auto-set `status = uploaded` on form submission |
| Search records returning 0 results | Records were all `chunked` or `error` — no `uploaded` records existed. Fixed by resetting a test record to `uploaded` |
| Workflow stopping at Search records node | Filter formula only matched `uploaded` but test records had other statuses — resolved by resetting record manually |

---

## Known Limitations

| Limitation | Detail |
|------------|--------|
| Short answer chunking | Very short documents (under 50 characters) are rejected entirely. There is no partial chunking for borderline-length content. |
| PDF text extraction | PDF uploads go through the Extract from File node. Complex or scanned PDFs may not extract cleanly. |
| Manual workflow trigger | The n8n workflow currently runs on manual trigger, not on a schedule. Records sit as `uploaded` until the workflow is manually executed. |
