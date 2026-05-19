I'll mark the interview collection complete and start producing the gap-analysis report.

Starting: *Produce Checkpoint 2 gap analysis report* (3/3)

**Checkpoint 2 Readiness Assessment**

**Status:** AT RISK

**What's Working:**  
- **Ingestion:** Extracts, cleans, and chunks text from text-based PDFs/notes.  
- **AI Core:** Generates true/false and short-answer questions reliably; writes to `Questions`.  
- **Specialist:** Quiz delivery, response storage, grading, and `Performance` entries function.  
- **Handoffs:** Documents → Questions and Questions → Responses have been tested end-to-end when workflows are executed.

**Critical Gaps (must fix before Checkpoint 2):**  
- **Automate n8n execution (owner: Ryan):** Replace manual run with an Airtable-triggered webhook or enable n8n cloud scheduler. Steps: (1) change n8n trigger to "Webhook" or "Airtable watch records", (2) create an Airtable Automation that calls the webhook when `Documents.status` becomes `chunked`/`ready`, (3) test with one record and confirm `Questions` are generated automatically.  
- **MCQ reliability validation & fallback (owner: Maryem):** Add post-processing in the Flowise chain to assert all `option_a`..`option_d` are non-empty; if any are blank/malformed, re-prompt the LLM with a constrained template or synthesize distractors from document text; set a confidence threshold and log failures.  
- **Ensure consistent scoring/feedback fields (owner: Noeleen):** Enforce population of `score_awarded`, `is_correct`, `graded_at`, and `overall_feedback` after grading. Steps: add validators in Specialist workflow and set defaults (e.g., `score_awarded=0`) so downstream reporting never sees nulls.  
- **Activate structured error logging (owner: Maria / Integration):** Have n8n and Flowise write failures to the `Errors` table with `error_message`, `error_type`, `timestamp`, and `source_record`. Add a final catch node in each workflow that creates an `Errors` record.  
- **Resolve `Quiz Results` vs `Performance` overlap (owner: Integration / Maria):** Decide single source-of-truth (prefer `Performance`) and implement a small mapping step: backfill existing `Quiz Results` into `Performance` and update all workflows to write to `Performance` only.

**Schema Issues Found:**  
- **Overlap:** `Quiz Results` duplicates `Performance` responsibilities — consolidate to `Performance`.  
- **Unused/placeholder fields:** `Errors` exists but is not written to; `question_count`, `confidence_score`, `generation_id`, and some performance feedback fields are inconsistently populated.  
- **Fields to change/update:**  
  - Standardize `question_count` usage: ensure `Quizzes.question_count` is set when quiz is created.  
  - Enforce `confidence_score` numeric range and optional default (0) in `Questions`.  
  - Ensure `Questions.option_a`..`option_d` are nullable=false at validation layer (not Airtable schema change required, but validated in AI Core).  
  - Use `Performance` as the canonical reporting table and remove/repurpose `Quiz Results` to avoid confusion.

**Recommended Fix Order (each item completable < 2 hours):**  
1. Automate n8n trigger via Airtable webhook or scheduler (Ryan).  
2. Add MCQ post-generation validation + simple distractor fallback in Flowise (Maryem).  
3. Add validators/defaults for grading fields in Specialist workflow (`score_awarded`, `is_correct`, `graded_at`) (Noeleen).  
4. Add error-capture node to n8n/Flowise writing to `Errors` (Maria).  
5. Define and apply `Quiz Results` → `Performance` mapping; update workflows to write to `Performance` only (Integration/Maria).

**Test Data Gaps:**  
- **Scanned (image) PDF:** Add `Documents` record title "scanned_exam_page.pdf", `status`: `uploaded`, `file_upload`: (image PDF) — expected `clean_text` empty or OCR-failure path.  
- **Empty upload:** `title`: "empty_upload.pdf", `clean_text`: "" — exercise failure handling.  
- **Very short notes:** `title`: "one_line_notes.txt", `clean_text`: "Key term: mitochondria." — tests short-source behavior.  
- **Very long document:** `title`: "long_chapter.txt", `clean_text`: (50k+ chars) — tests chunking, timeouts.  
- **Malformed file:** `title`: "corrupt.pdf" — tests error logging.  
- **MCQ edge case document:** `title`: "mcq_broken_source.txt" — a doc that previously produced blank options; `Questions` should show `mcq` and trigger MCQ fallback.  
- **Bad-format text / non-English:** `title`: "foreign_or_noise.txt" — tests generator robustness.  
For each test record add expected `Documents.status` transitions and an assertion checklist: does `Questions` appear, are all `option_a`..`option_d` populated for `mcq`, is a `Performance` record created after grading, and is an `Errors` record created for failures.

Would you like me to (A) generate an Airtable test-data CSV with example rows for these cases, (B) draft the exact n8n change steps (Webhook + Airtable Automation) for Ryan to apply, or (C) create a short Flowise post-processing prompt-template to enforce MCQ option generation?
