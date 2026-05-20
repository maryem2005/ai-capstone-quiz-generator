# Prompt Log — Maryem Elgebaly
 
**Project:** AI-Powered Quiz Generator  
**Team:** ai-capstone-quiz-generator (Maria, Ryan, Maryem)  
**My Component:** Question Generator (AI Core)  
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
