# Prompt Log — Maryem Elgebaly
 
**Project:** AI-Powered Quiz Generator  
**Team:** ai-capstone-quiz-generator (Maria, Ryan, Maryem)  
**My Component:** Question Generator (AI Core) & Quiz Delivery & Scoring
**AI Tools Used:** GitHub Copilot
 
---
 
## Entry 1 — 2026-04-24 — Capstone Audit Interview
 
**Context:** Had copilot-instructions.md open in VS Code with full project context loaded. Running Week 8 lab capstone audit before Checkpoint 2.
 
**Prompt:**
> I need you to act as a capstone project advisor for a university AI integration course. Interview me about my project's current state and produce a structured gap analysis.
 
**Result:** Copilot interviewed me with 10 questions one at a time and produced a full Checkpoint 2 Readiness Assessment. Status: AT RISK. Identified 4 critical gaps including my unbuilt Step 7 workflow and Noeleen's unstarted component.
 
**Evaluation:** Accurate and honest. The gaps it identified were real — especially Step 7 being unbuilt and no end-to-end testing done yet.
 
**What I changed:** Nothing — the report was accurate as generated.
 
**What I learned:** Having copilot-instructions.md loaded made Copilot extremely specific to our actual project. It referenced real field names like document_id, options_json, and raw_text instead of giving generic advice. Specificity in the instructions file = specificity in the output.
 
---
 
## Entry 2 — 2026-04-25 — Component README Generation
 
**Context:** copilot-instructions.md loaded in VS Code. Asked Copilot to generate a README for my component.
 
**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Question Generator component. Include what it does, how it connects to other components, setup instructions, how to test it, and known limitations.
 
**Result:** Generated a complete README with actual field names, table names, setup instructions for Groq and n8n, test steps, and known limitations including the document_id linked record risk and options_json parsing issue.
 
**Evaluation:** Excellent. Referenced our actual schema and tools correctly. Known limitations section was especially useful — identified real risks we needed to address before Checkpoint 2.
 
**What I changed:** Added it to the bottom of my existing README.md to preserve Week 4 and Week 5 history.
 
**What I learned:** The more specific your instructions file, the more useful the generated artifact. Generic prompts give generic READMEs — project-specific context gives project-specific output.
 
---
 
## Entry 3 — 2026-04-27 — Error Handling for Groq API Failures
 
**Context:** My n8n Question Generator workflow had no error handling — if Groq API went down, the whole workflow crashed silently with no record of what failed.
 
**Prompt:**
> Help me add error handling to my n8n workflow for when the Groq API fails
 
**Result:** Added an If node to detect API errors and an Update record node to write status = "error" to the Documents table in Airtable. Workflow now catches failures gracefully instead of crashing.
 
**Evaluation:** Worked well. The error path now logs failures so they're visible in Airtable instead of disappearing silently.
 
**What I learned:** Error handling should be built in from the start, not added after. Having a visible error state in Airtable made debugging much easier later in the project.
 
---
 
## Entry 4 — 2026-04-29 — Confidence-Based Routing Setup
 
**Context:** Need to implement confidence-based routing so that questions above a confidence threshold go forward and low-confidence questions go to a review queue.
 
**Prompt:**
> Help me set up confidence-based routing using question difficulty in my IF node
 
**Result:** Updated the IF node to route hard questions to a reviewed path and easy/medium questions to the generated path. Questions now routed based on difficulty level automatically.
 
**Evaluation:** Successfully routes questions based on difficulty. The logic is simple but effective — difficulty acts as a proxy for confidence since Groq assigns it per question.
 
**What I learned:** Routing logic doesn't need to be complex to be useful. A single IF node with a clear condition is easier to debug and maintain than a complicated multi-branch setup.
 
---
 
## Entry 5 — 2026-04-30 — Airtable Dashboard Views
 
**Context:** Project needed dashboard views for Checkpoint 2 to show system health and pipeline status at a glance.
 
**Prompt:**
> Help me create Airtable dashboard views for error monitoring and pipeline status
 
**Result:** Created an Error Monitor view filtered by status = "error" and a Pipeline Status view grouped by status field. Dashboard now shows system health at a glance.
 
**Evaluation:** Views work well. Error Monitor is especially useful during testing — you can immediately see which records failed without manually scanning the table.
 
**What I learned:** Building dashboard views early makes debugging faster. Being able to see pipeline status grouped visually saved a lot of time during Checkpoint 2 testing.
 
---
 
## Entry 6 — 2026-05-09 — Fixing True/False Grading Case Sensitivity Bug
 
**Context:** During testing, True/False questions were being marked wrong even when the answer was correct. A user typing "false" (lowercase) failed because the correct answer was stored as "False" (capital F). This broke grading for an entire question type.
 
**Prompt:**
> My IF node is marking True/False answers as wrong even when they match. The submitted answer is lowercase but the stored correct answer has a capital letter. How do I fix the case mismatch in n8n?
 
**Result:** Fixed by applying .toLowerCase() to both the submitted_answer and correct_answer fields inside the IF node expression so comparison is always case-insensitive:
```
{{ $('Airtable Trigger').item.json.fields.submitted_answer.toLowerCase() }}
{{ $('Get a record1').item.json.fields.correct_answer.toLowerCase() }}
```
 
**Evaluation:** Fix worked immediately. Record 68 (holasss) went from incorrect to correct after the fix. All True/False questions now grade accurately regardless of capitalization.
 
**What I learned:** Always normalize string comparisons before evaluating them. Case sensitivity bugs are easy to miss during happy-path testing but break real user submissions instantly.
 
---
 
## Entry 7 — 2026-05-11 — Fixing Response Type Hardcoding
 
**Context:** The response_type field in the Responses table was always showing "multiple_choice" regardless of the actual question type. It was hardcoded in both the TRUE and FALSE branch Update record nodes.
 
**Prompt:**
> My response_type field is always saving as multiple_choice even for True/False questions. How do I make it dynamic so it pulls the actual question type from the Questions table?
 
**Result:** Changed the response_type field in both Update record nodes from a hardcoded dropdown to a dynamic expression:
```
{{ $('Get a record1').item.json.fields.question_type }}
```
True/False questions now correctly show true_false and MCQ questions show mcq.
 
**Evaluation:** Fix worked perfectly. Data integrity in the Responses table is now accurate across all question types.
 
**What I learned:** Hardcoded values in workflow nodes are easy to miss and cause silent data quality issues. Always check whether dropdown values should actually be dynamic expressions pulling from upstream data.
 
---
 
## Entry 8 — 2026-05-12 — Debugging Performance Table Dynamic Matching
 
**Context:** The Performance table workflow was hardcoded to only update record 12 (Maryem_WH). It wasn't dynamically finding the correct Performance record for each user, so 14 out of 15 users got no Performance record created.
 
**Prompt:**
> My Performance workflow is only updating one hardcoded record instead of finding the right record for each user dynamically. The Find Performance Record node returns 0 results for most users. How do I fix dynamic matching in n8n with Airtable?
 
**Result:** Applied 5 fixes: updated Airtable Trigger formula to filter by user_id, updated Search records filter to match current user, replaced Update record with Create or Update (upsert) node matching on user_id, bypassed the broken Find Performance Record node, and fixed attempt_id expression. All 15 users now get correct Performance records dynamically.
 
**Evaluation:** This was the hardest bug in the entire project. The root cause was a combination of a broken filter formula, a hardcoded node, and a trigger that wasn't scoped to the current user. Fixing all 5 pieces together resolved it completely.
 
**What I learned:** When a workflow produces wrong results for most users but correct results for one, the bug is almost always in the matching/filtering logic, not the update logic. Start by auditing how the workflow identifies which record to update.
 
---
 
## Entry 9 — 2026-05-14 — End-to-End Pipeline Verification 
 
**Context:** After all fixes were applied, needed to run one clean end-to-end test tracing a single record through the entire pipeline from Documents → Questions → Quizzes → Responses → Performance.
 
**Prompt:**
> Help me design a clean end-to-end test for my quiz generator pipeline. What should I check at each stage to confirm the full flow is working?
 
**Result:** Ran a full trace using Document Record 8 (World War II, World History). Questions 938-943 generated correctly. Response Record 60 created for user Step12Test, auto-graded within 1 minute. Performance Record 33 created automatically with all fields populated including weak_topics: American History.
 
**Evaluation:** Full pipeline confirmed working end-to-end. Every field that was previously broken now populates correctly for new records.
 
**What I learned:** End-to-end testing with a fresh user and fresh record is much more reliable than checking individual nodes in isolation. Integration bugs only show up when the full chain runs together.
 
---
 
## Entry 10 — 2026-05-17 — Portfolio README Generation
 
**Context:** Week 13 portfolio assembly — needed to create a professional GitHub profile README that presents my work clearly to employers and professors.
 
**Prompt:**
> Write me a professional GitHub profile README for my personal profile. I'm an AI Integration Specialist, I built the Question Generator component for an AI-Powered Quiz Generator capstone at John Jay College using n8n, Flowise, Airtable, and Groq.
 
**Result:** Generated a complete profile README with personal intro, featured project section with component description and repo link, tools list, currently learning section, and contact info. Deployed to the maryem2005/maryem2005 special GitHub repo so it shows at the top of my profile.
 
**Evaluation:** Clean and professional. Much better than having an empty profile. The featured project section directly links to my component work.
 
**What I learned:** Your GitHub profile is often the first thing a recruiter sees. Having a profile README that explains what you build and links to real work makes a much stronger impression than a list of repo names with no context.
 ## Entry 11 — 2026-05-19 — Diagnosing the Airtable Trigger Cursor Bug
 
**Context:** After replacing the broken Airtable Trigger with a Schedule Trigger + Search Records, new form submissions (records 80, 81, 82) were not being picked up by the grading workflow even though the workflow was active and running every minute. Multiple attempts to reset the status field made no difference.
 
**Prompt:**
> My n8n grading workflow is running every minute and succeeding, but it keeps re-processing the same old record (record 78) instead of picking up new submissions. Records 80, 81, 82 all have blank status but never get processed. What is happening?
 
**Result:** Diagnosed the root cause as n8n's internal cursor mechanism. The Airtable Trigger tracks records using `createdTime` internally — once it logs a record as "seen," it will not revisit it even if the record's fields are still blank. Records 80-82 were created during a moment when the trigger polled but the records were not yet fully saved, so the cursor advanced past them permanently. Switching the trigger to Schedule + Search Records with a formula filter (`{status} = BLANK()`) resolved this entirely — the Search Records node queries Airtable fresh on every run without any cursor state.
 
**Evaluation:** Correct diagnosis. After the trigger swap, the first fresh submission (record 85) was picked up and fully graded within 60 seconds with all fields populated.
 
**What I learned:** The Airtable Trigger node in n8n is stateful — it maintains an internal cursor and will never go back for records it has already seen, even if those records are incomplete. For polling-based workflows where records might be created before all fields are populated, a Schedule Trigger + Search Records combination is more reliable because it has no cursor and queries Airtable's current state on every run.
 
---
 
## Entry 12 — 2026-05-19 — Fixing Node Reference Errors After Trigger Replacement
 
**Context:** After replacing the Airtable Trigger with Schedule + Search Records, every node downstream that referenced `$('Airtable Trigger')` broke with "Referenced node doesn't exist" errors. This affected the IF node, both Update record nodes, and the Create or update a record node — essentially the entire workflow.
 
**Prompt:**
> I replaced my Airtable Trigger with a Schedule Trigger and Search Records node. Now every node in my workflow has red errors saying "Referenced node doesn't exist." What do I need to update and what are the correct expressions?
 
**Result:** Systematically replaced every `$('Airtable Trigger')` reference across all nodes with `$('Search records')`. Key expressions updated:
 
```
id field:         {{ $('Search records').item.json.id }}
submitted_answer: {{ $('Search records').item.json.fields.submitted_answer }}
quiz:             {{ [$('Search records').item.json.fields.quiz[0]] }}
user_id:          {{ $('Search records').item.json.fields.user_id }}
performance:      {{ [$('Search records').item.json.fields.performance[0]] }}
```
 
The IF node comparison was also updated from `$('Airtable Trigger')` to `$('Search records')` for the submitted_answer field.
 
**Evaluation:** All red errors cleared after updating the references. The workflow ran successfully and graded record 85 (test_2, Photosynthesis) correctly on the first execution.
 
**What I learned:** When replacing a trigger node in n8n, every downstream node that referenced the old trigger by name will break. It is worth doing a full audit of every expression in the workflow before re-running — not just fixing nodes one at a time as errors appear. Searching for the old node name string across all nodes at once is faster than discovering them one by one during execution.
 
---
 
## Entry 13 — 2026-05-19 — Fixing Duplicate Records with Upsert
 
**Context:** Every time the Quiz Question Assignment workflow and the Document Ingestion workflow ran, they created brand new records in the Quizzes and Questions tables instead of updating existing ones. This resulted in duplicate quizzes and duplicate questions accumulating with every run.
 
**Prompt:**
> My n8n workflows keep creating duplicate records in Airtable every time they run. The Create Quiz node creates a new quiz even if one with that topic already exists. Same with questions. How do I prevent duplicates without deleting old records manually after every run?
 
**Result:** Changed the operation on both nodes from `Create` to `Upsert` and set the matching fields:
- Quiz node: match on `topic`
- Questions node: match on `question_text`
With upsert, n8n checks whether a record with the matching field value already exists before writing. If it exists, the record is updated. If it does not exist, a new record is created. Running the workflows multiple times no longer creates duplicates.
 
**Evaluation:** Confirmed working — ran both workflows twice in a row and the record counts in Quizzes and Questions tables stayed the same. Previously the counts would grow by 5 and 29 respectively on every run.
 
**What I learned:** Any workflow that creates records on a schedule or on repeated triggers should use upsert instead of create by default. The match field needs to be something unique per record — `topic` works for quizzes because there is one quiz per topic, and `question_text` works for questions because question text is unique. Using `generation_id` as the match field did not work because multiple questions share the same generation ID.
 
---
 
## Entry 14 — 2026-05-19 — Updating the Flowise Prompt to Generate 10 Questions
 
**Context:** The system was only generating 3 questions per document chunk. The project specification required at least 10 questions with a defined type mix (minimum 4 MCQ, 3 true/false, 3 short answer). The Flowise Human Message field needed to be updated.
 
**Prompt:**
> My Flowise chain is only generating 3 questions per document. I need it to generate at least 10 with a specific mix of question types. How should I update the prompt?
 
**Result:** Updated the Human Message field in the Chat Prompt Template node in Flowise from:
 
```
Generate 3 quiz questions from this study material: {question}
```
 
To:
 
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
 
Kept `{question}` at the end to preserve the variable that injects the study material content into the prompt.
 
**Evaluation:** Confirmed working on the Chemistry and Sociology documents — both generated 3 questions correctly after the update, with the type mix matching the required distribution. The existing JavaScript Code node parsed all question types without modification.
 
**What I learned:** Prompt changes in Flowise take effect immediately without any changes to the n8n workflow. Keeping the Flowise chain as a separate layer from n8n is valuable precisely because of this — the prompt can be tuned independently without touching the automation logic. Also learned that `{question}` must always remain in the prompt or the model receives no study material to generate from.

---


# Prompt Log — Quiz Delivery & Scoring
**Component:** Quiz Delivery & Scoring  
**Workflows:** Quiz Question Assignment · Quiz Delivery & Scoring Step 14 · Quiz Delivery & Scoring Step 15
 
---
 
## Entry 1 — Building the Per-Question Grading Workflow (Step 14)
 
**Date:** May 9, 2026  
**Step:** Step 14 — Quiz Delivery & Scoring Skeleton Build
 
**Prompt / Approach:**  
Designing the n8n grading workflow from scratch. The problem was that Airtable's free plan has no native webhook, so there was no way to trigger n8n instantly on form submission. I needed a polling approach that would still grade responses quickly.
 
**Context:**  
A student submits a quiz answer through the Airtable form. That creates a new record in the Responses table. n8n needs to detect that new record, fetch the correct question, compare the answers, and write the grading result back — all automatically.
 
**What I built:**  
Set up an Airtable Trigger polling every minute with the formula `{status} = BLANK()`. This catches every new ungraded submission. Added a 5-second Wait node after the trigger because without it, n8n would read the record before Airtable had finished saving all the form fields — the trigger was only returning 2 fields instead of the full record.
 
The IF node compared `submitted_answer` directly against `correct_answer` from the Questions table. TRUE branch marks `is_correct` ON and sets `score_awarded: 1`. FALSE branch marks `is_correct` OFF, sets `score_awarded: 0`, and pulls `missed_topic` from the question's `topic` field.
 
**Evaluation:**  
Worked on the first full test. Records 41, 47, and 48 all graded correctly within 1 minute of submission with no manual intervention. The Wait node was critical — removing it caused the trigger to only return partial data.
 
**What I learned:**  
Polling triggers need a buffer between trigger and first action when the data source (Airtable form) is slower than the automation. 5 seconds was enough. Also learned that linked record fields in Airtable come back as arrays — had to use `[0]` to extract the Question record ID from `fields.question`.
 
---
 
## Entry 2 — Fixing the Get a Record1 Table Mismatch
 
**Date:** May 10, 2026  
**Step:** Debugging — Cell Biology responses not grading
 
**Prompt / Approach:**  
Diagnosing why the grading workflow ran but always routed to the FALSE branch, even when the correct answer was submitted.
 
**Context:**  
Records 72 and 73 were grading fine. Record 74 (Cell Biology) was not. The trigger was picking it up, the Wait node was passing, but every single response was being marked incorrect regardless of what was submitted.
 
**What I investigated:**  
Traced each node output manually. The Airtable Trigger output looked fine. The Wait node passed. Then checked Get a record1 — the node was returning unexpected data. The Record ID expression was set to `$('Airtable Trigger').item.json.id`, which is the Response record's own Airtable ID. But the node's Table was set to Questions. So n8n was searching the Questions table for a record with an ID that belonged to the Responses table — either failing silently or returning a completely wrong record.
 
**Fix applied:**
```javascript
// Before (wrong — using the Response record's own ID):
$('Airtable Trigger').item.json.id
 
// After (correct — using the linked Question record ID from the Response):
{{ $('Airtable Trigger').item.json.fields.question[0] }}
```
 
**Evaluation:**  
Immediate fix. After this change, Get a record1 returned the actual Question record with `correct_answer`, `explanation`, `question_type`, and `topic` all populated. The IF node started routing correctly and grading worked.
 
**What I learned:**  
Always verify that the record ID expression in a Get a Record node matches the table it's pointing to. A response record ID will not find anything in the Questions table. n8n does not throw an obvious error in this case — it either returns NOT_FOUND silently or returns data from the wrong record, both of which produce silent incorrect behavior downstream.
 
---
 
## Entry 3 — Airtable Automation Conflicting with n8n Trigger
 
**Date:** May 18, 2026  
**Step:** Debugging — New form submissions not being picked up by grading workflow
 
**Prompt / Approach:**  
Investigating why new Cell Biology submissions were not triggering the grading workflow even though the trigger formula looked correct.
 
**Context:**  
The grading trigger used `{status} = BLANK()` to catch new ungraded responses. Fresh form submissions should have an empty status field, so the trigger should pick them up on the next poll. But new records were being skipped entirely.
 
**What I found:**  
Airtable had a native Automation (Automation 1) configured to fire "When a form is submitted." This automation was immediately writing to the `status` field on every new Response record. Since n8n polls on a 1-minute interval, by the time n8n checked, `status` was no longer blank — it had already been filled by Airtable's automation. n8n's filter saw a non-blank status and skipped the record.
 
**Fix applied:**  
Turned off Airtable Automation 1.
 
**Evaluation:**  
Worked immediately. New submissions were picked up on the next n8n poll cycle. This also explained why earlier tests sometimes worked and sometimes did not — Automation 1 was toggled on and off at different points during development, which made the grading seem inconsistent without an obvious reason.
 
**What I learned:**  
When two automation systems (Airtable native automations and n8n) operate on the same table and same fields, they will race each other. Airtable's native automations fire faster than n8n's polling interval. Always check whether any Airtable automations are touching the same fields that n8n is filtering on before debugging the n8n workflow.
 
---
 
## Entry 4 — IF Node Routing Everything to False Branch
 
**Date:** May 18, 2026  
**Step:** Debugging — All responses marked incorrect after fixing Get a Record1
 
**Prompt / Approach:**  
After fixing the Get a record1 table mismatch, the workflow ran without errors but all answers were still being marked wrong regardless of whether they were correct.
 
**Context:**  
The IF node was comparing the submitted answer against the correct answer. Get a record1 was now returning the right Question record. But the IF node's second expression was still pointing to the wrong field.
 
**What I found:**  
The IF node's second expression was `correct_answer_lookup[0].toLowerCase()`. This was a lookup field from the Response record — a field that pulled `correct_answer` from the linked Question using Airtable's lookup field feature. It returned an array. But now that Get a record1 was correctly returning the Question record, the right field to use was simply `correct_answer` — a plain string directly on the Question record.
 
**Fix applied:**
```javascript
// Before (pulling lookup array from Response record):
correct_answer_lookup[0].toLowerCase()
 
// After (pulling plain string from Question record via Get a record1):
$('Get a record1').item.json.fields.correct_answer.toLowerCase()
```
 
**Evaluation:**  
Fixed the routing. Correct answers now route to the TRUE branch and incorrect ones route to FALSE. The `.toLowerCase()` normalization on both sides also fixed the earlier case sensitivity issue where `false` (lowercase) would fail against `False` (capital F).
 
**What I learned:**  
After restructuring which node a field comes from, every downstream expression that references that field needs to be updated too. Fixing Get a record1 meant all field references in the IF node and both Update record nodes also needed to be re-pointed to `$('Get a record1')` rather than their old sources.
 
---
 
## Entry 5 — Fixing All Field Expressions in Update Record Nodes
 
**Date:** May 19, 2026  
**Step:** Debugging — `feedback`, `response_type`, `missed_topic` not populating
 
**Prompt / Approach:**  
After fixing the IF node routing, grading was working but several fields on the Response record were still empty after grading: `feedback`, `response_type`, and `missed_topic`.
 
**Context:**  
These fields were mapped in both the TRUE and FALSE branch Update record nodes. The expressions had been written before the Get a record1 fix, so they were pointing to the old field sources.
 
**What I found and fixed:**
 
`feedback` was mapped to `fields.explanation` but from the Response record, which has no `explanation` field. Fixed to:
```javascript
{{ $('Get a record1').item.json.fields.explanation }}
```
 
`response_type` was hardcoded to `multiple_choice` as a dropdown selection. Fixed to a dynamic expression:
```javascript
{{ $('Get a record1').item.json.fields.question_type }}
```
 
`missed_topic` was mapped correctly on the FALSE branch already. Confirmed it pulled from:
```javascript
{{ $('Get a record1').item.json.fields.topic }}
```
 
`performance` was mapped in both branches but does not exist at grading time — it gets written back later by the Performance Calculator. Deleted from both Update record nodes to stop the field mapping error.
 
**Evaluation:**  
All fields populated correctly after the fix. `feedback` now shows the question explanation, `response_type` correctly shows `true_false` or `multiple_choice` depending on the question, and `missed_topic` shows the subject area for wrong answers.
 
**What I learned:**  
When fixing one node in a chain, audit all downstream field expressions. The Update record nodes had been written assuming a different data source — fixing Get a record1 made all of those expressions stale at once.
 
---
 
## Entry 6 — Writing Performance ID Back to Response Records
 
**Date:** May 19, 2026  
**Step:** Debugging — `performance` field not linking on Response records
 
**Prompt / Approach:**  
After removing `performance` from the grader nodes (since the Performance record doesn't exist at grading time), Response records no longer had a `performance` link. The connection only existed in one direction — Performance had a `responses` array pointing to Response records, but Responses had no `performance` field filled.
 
**Context:**  
The Performance Calculator creates or updates the Performance record and writes an array of Response record IDs into the `responses` field. But Airtable linked record fields are not automatically bidirectional in n8n — setting `responses` on Performance does not automatically set `performance` on each Response.
 
**Fix applied:**  
Added a new Update record node at the very end of the Performance Calculator workflow, after the Create or Update node:
 
```
Node:        Update record
Table:       Responses
Record ID:   {{ $('Create or update a record').item.json.fields.responses[0] }}
performance: {{ [$('Create or update a record').item.json.id] }}
```
 
The `performance` field is a linked record field in Airtable so the value must be passed as an array, not a plain string.
 
**Evaluation:**  
This completed the bidirectional link. Response records now show the correct linked Performance record when opened in Airtable, and clicking through from a Response to its Performance record works correctly.
 
**What I learned:**  
Linked record fields in Airtable require explicit write-back in both directions when using n8n. Creating a Performance record with a `responses` array does not backfill the `performance` field on those Response records — that has to be a separate node. Always wrap linked record ID values in an array `[ ]` or Airtable will reject the field update.
 
---
 
## Entry 7 — Performance Table Upsert to Fix Hardcoded Record Matching
 
**Date:** May 12, 2026  
**Step:** Step 10 — Dynamic Performance Record Matching
 
**Prompt / Approach:**  
The Performance Calculator was hardcoded to update only record 12 (Maryem_WH / World History). Every other user's quiz submission either failed or overwrote the wrong record. The original design used a "Find Performance Record" search node to look up the right record dynamically, but `attempt_id` was coming through empty in the grouped results, making the match unreliable.
 
**Context:**  
The workflow needed to find the correct Performance record for any user/quiz combination and update it, or create a new one if none existed. The search-then-update pattern was fragile because the search sometimes returned multiple records, and the match condition depended on `attempt_id` which wasn't reliably populated.
 
**What I did:**  
Replaced the entire Update record node with a "Create or Update" (upsert) node. Set the match fields to `user_id + quiz` instead of relying on a record ID lookup. Bypassed the Find Performance Record search node entirely by connecting the IF node's true branch directly to the upsert node.
 
**Evaluation:**  
Confirmed working across 15 users — Maryem_WH (50%, failed), Maryem_test5 (100%, passed), Maryem (100%, passed), and Step12Test (0%, failed, weak_topics: American History). No hardcoded record IDs anywhere in the workflow.
 
**What I learned:**  
Upsert with natural key matching is more reliable than search-then-update when the lookup criteria is unstable. `user_id + quiz` is a stable unique combination for this use case — there should only ever be one Performance record per user per quiz. Using it as the upsert key eliminates the need to find the record first.
 
---
 
## Entry 8 — Duplicate Quiz Creation in Quiz Question Assignment
 
**Date:** May 17, 2026  
**Step:** Quiz Question Assignment — Deduplication Problem
 
**Prompt / Approach:**  
The Quiz Question Assignment workflow was running every minute and creating a new quiz record for every document on every run. After a few minutes there were 20+ duplicate Biology quizzes in the Quizzes table.
 
**Context:**  
The original workflow searched Documents (22 records) and created a new Quiz record for each one on every trigger run. There was no check for whether a quiz already existed for that document.
 
**What I tried first:**  
Added a "Check Existing Quiz" node filtering by `AND({document} = "...", {status} = "active")`. This failed because `document` in the Quizzes table is a linked record field — Airtable's filter formula cannot match a linked record field with a plain text string. Tried `{quiz_title} = "{{ $('Search Documents').item.json.fields.topic }} Practice Quiz"` next — this returned 0 results because the expression rendered incorrectly inside the filter formula, stopping the workflow before it could create anything useful.
 
**What I did instead:**  
Completely rebuilt the workflow with a different starting point. Instead of starting from Documents, start from Questions and group by topic:
 
```
Schedule Trigger → Search Questions → Code in JavaScript → Create Quiz → Update Record
```
 
The Code node groups all questions with `status = generated` by topic and returns one output item per topic. This means the workflow processes one quiz per topic per run, not one per document.
 
Also had to fix the Update record node — `questionIds` was `undefined` because `{{ $json.questionIds }}` was referencing the previous node's output, not the Code node specifically. Fixed to `{{ $('Code in JavaScript').item.json.questionIds }}`.
 
**Evaluation:**  
Workflow found 320 questions, grouped into 2 topics (World History + Algebra), created 2 quizzes, and linked all questions correctly. No duplicates.
 
**What I learned:**  
Deduplication via filter is fragile with Airtable linked record fields. Restructuring the workflow so duplicates cannot occur in the first place is more reliable than trying to detect and skip them. Also: always use the explicit node name reference `$('Node Name')` when the expression references data from a node that is not the immediately previous one — `$json` only refers to the last node's output.
 
---
 
## Entry 9 — Fixing the Trigger Stuck on Old Records
 
**Date:** May 18, 2026  
**Step:** Debugging — Grading trigger not moving past old broken records
 
**Prompt / Approach:**  
The Airtable Trigger in the grading workflow was stuck on record 36 — an old response record from May 9th — and refused to process any newer submissions.
 
**Context:**  
n8n's Airtable Trigger works by remembering the last record it processed and only picking up newer ones. Record 36 had no `question` field linked. Every time the trigger ran, it tried to process record 36, Get a record1 failed with NOT_FOUND because there was no question to fetch, and the workflow errored out without advancing its internal cursor to newer records.
 
**What I tried:**  
Changed the trigger formula to `AND({status} = BLANK(), {record_id} = 71)` to force it past record 36. The trigger still returned record 36 because n8n's cursor was stuck. Changed the Trigger Field from `created_at` to `record_id` — partial improvement, the trigger moved to record 69, then got stuck on other old records with the same problem (no question field linked).
 
**Fix applied:**  
Deleted all old Response records that had no `question` field linked. Kept only records 72, 73, and 74 which were properly structured with question links.
 
**Evaluation:**  
After cleanup the trigger moved to record 72 and processed it correctly. The grading workflow ran end to end for the first time on a fresh submission.
 
**What I learned:**  
n8n's Airtable Trigger advances its cursor only when a workflow execution completes successfully. If a downstream node fails on every run, the cursor never moves and the trigger is permanently stuck on that record. The cleanest solution is to delete or fix the blocking record rather than trying to force the trigger past it with formula filters.
 
---
 
## Entry 10 — Full End-to-End Pipeline Test (May 18, 2026)
 
**Date:** May 18, 2026  
**Step:** Full demo pipeline test
 
**Prompt / Approach:**  
After all individual fixes were confirmed, ran a clean end-to-end test to verify the entire Quiz Delivery pipeline worked from form submission through to a completed Performance record.
 
**Context:**  
Submitted Photosynthesis study notes through Ryan's document ingestion form. This was the first time the entire pipeline was tested with a brand new document that had never been through the system before.
 
**What was confirmed at each stage:**
 
Document ingestion: Photosynthesis notes chunked automatically by Ryan's workflow.
 
Question generation: Flowise generated Photosynthesis questions from the chunks. Questions written to the Questions table with `status: generated`, `topic: Photosynthesis`.
 
Quiz assignment: The Quiz Question Assignment workflow detected the new Photosynthesis questions, created Photosynthesis Practice Quiz (record 111), and linked the questions automatically. No manual intervention.
 
Form submission and grading: Submitted a quiz answer through the Airtable form. The grading workflow (Step 14) picked up the new Response record within 1 minute, fetched the correct Question record, compared answers, and wrote `is_correct`, `score_awarded`, `feedback`, `response_type`, `submitted_at`, `graded_at`, and `status: graded` correctly.
 
Performance calculation: The Performance Calculator (Step 15) detected the graded Response, calculated `score_percent`, `passed`, `weak_topics`, and created Performance record 47. The write-back node then updated the Response record's `performance` field with the Performance record ID.
 
**Evaluation:**  
Full pipeline confirmed working end to end for a completely new document with no manual intervention at any stage. Record 72 (user: Mary, Photosynthesis) graded correctly with `score_awarded: 1.0` and Performance record 47 created automatically. Record 73 (user: panda2.0, World History) also graded correctly with Performance record 51 created.
 
**What I learned:**  
Testing with a brand new document that has never been in the system is a better end-to-end test than re-running the same records. It exercises the full pipeline including quiz creation, which re-tests would skip since the quiz already existed. The Airtable Automation conflict (Entry 3) was found precisely because this test used fresh records that triggered the automation.
