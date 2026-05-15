# Prompt Log — Maryem Elgebaly
 
**Project:** AI-Powered Quiz Generator  
**Team:** ai-capstone-quiz-generator (Maria, Ryan, Maryem)  
**My Component:** Question Generator (AI Core)  
**AI Tools Used:** GitHub Copilot, Claude  
 
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
 
## Entry 2 — 2026-04-24 — Component README Generation
 
**Context:** copilot-instructions.md loaded in VS Code. Asked Copilot to generate a README for my component.
 
**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Question Generator component. Include what it does, how it connects to other components, setup instructions, how to test it, and known limitations.
 
**Result:** Generated a complete README with actual field names, table names, setup instructions for Groq and n8n, test steps, and known limitations including the document_id linked record risk and options_json parsing issue.
 
**Evaluation:** Excellent. Referenced our actual schema and tools correctly. Known limitations section was especially useful — identified real risks we needed to address before Checkpoint 2.
 
**What I changed:** Added it to the bottom of my existing README.md to preserve Week 4 and Week 5 history.
 
**What I learned:** The more specific your instructions file, the more useful the generated artifact. Generic prompts give generic READMEs — project-specific context gives project-specific output.
 
---
 
## Entry 3 — 2026-04-24 — Error Handling for Groq API Failures
 
**Context:** My n8n Question Generator workflow had no error handling — if Groq API went down, the whole workflow crashed silently with no record of what failed.
 
**Prompt:**
> Help me add error handling to my n8n workflow for when the Groq API fails
 
**Result:** Added an If node to detect API errors and an Update record node to write status = "error" to the Documents table in Airtable. Workflow now catches failures gracefully instead of crashing.
 
**Evaluation:** Worked well. The error path now logs failures so they're visible in Airtable instead of disappearing silently.
 
**What I learned:** Error handling should be built in from the start, not added after. Having a visible error state in Airtable made debugging much easier later in the project.
 
---
 
## Entry 4 — 2026-04-24 — Confidence-Based Routing Setup
 
**Context:** Need to implement confidence-based routing so that questions above a confidence threshold go forward and low-confidence questions go to a review queue.
 
**Prompt:**
> Help me set up confidence-based routing using question difficulty in my IF node
 
**Result:** Updated the IF node to route hard questions to a reviewed path and easy/medium questions to the generated path. Questions now routed based on difficulty level automatically.
 
**Evaluation:** Successfully routes questions based on difficulty. The logic is simple but effective — difficulty acts as a proxy for confidence since Groq assigns it per question.
 
**What I learned:** Routing logic doesn't need to be complex to be useful. A single IF node with a clear condition is easier to debug and maintain than a complicated multi-branch setup.
 
---
 
## Entry 5 — 2026-04-24 — Airtable Dashboard Views
 
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
 
## Entry 7 — 2026-05-09 — Fixing Response Type Hardcoding
 
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
 
## Entry 8 — 2026-05-11 — Debugging Performance Table Dynamic Matching
 
**Context:** The Performance table workflow was hardcoded to only update record 12 (Maryem_WH). It wasn't dynamically finding the correct Performance record for each user, so 14 out of 15 users got no Performance record created.
 
**Prompt:**
> My Performance workflow is only updating one hardcoded record instead of finding the right record for each user dynamically. The Find Performance Record node returns 0 results for most users. How do I fix dynamic matching in n8n with Airtable?
 
**Result:** Applied 5 fixes: updated Airtable Trigger formula to filter by user_id, updated Search records filter to match current user, replaced Update record with Create or Update (upsert) node matching on user_id, bypassed the broken Find Performance Record node, and fixed attempt_id expression. All 15 users now get correct Performance records dynamically.
 
**Evaluation:** This was the hardest bug in the entire project. The root cause was a combination of a broken filter formula, a hardcoded node, and a trigger that wasn't scoped to the current user. Fixing all 5 pieces together resolved it completely.
 
**What I learned:** When a workflow produces wrong results for most users but correct results for one, the bug is almost always in the matching/filtering logic, not the update logic. Start by auditing how the workflow identifies which record to update.
 
---
 
## Entry 9 — 2026-05-11 — End-to-End Pipeline Verification (Step 12)
 
**Context:** After all fixes were applied, needed to run one clean end-to-end test tracing a single record through the entire pipeline from Documents → Questions → Quizzes → Responses → Performance.
 
**Prompt:**
> Help me design a clean end-to-end test for my quiz generator pipeline. What should I check at each stage to confirm the full flow is working?
 
**Result:** Ran a full trace using Document Record 8 (World War II, World History). Questions 938-943 generated correctly. Response Record 60 created for user Step12Test, auto-graded within 1 minute. Performance Record 33 created automatically with all fields populated including weak_topics: American History.
 
**Evaluation:** Full pipeline confirmed working end-to-end. Every field that was previously broken now populates correctly for new records.
 
**What I learned:** End-to-end testing with a fresh user and fresh record is much more reliable than checking individual nodes in isolation. Integration bugs only show up when the full chain runs together.
 
---
 
## Entry 10 — 2026-05-15 — Portfolio README Generation
 
**Context:** Week 13 portfolio assembly — needed to create a professional GitHub profile README that presents my work clearly to employers and professors.
 
**Prompt:**
> Write me a professional GitHub profile README for my personal profile. I'm an AI Integration Specialist, I built the Question Generator component for an AI-Powered Quiz Generator capstone at John Jay College using n8n, Flowise, Airtable, and Groq.
 
**Result:** Generated a complete profile README with personal intro, featured project section with component description and repo link, tools list, currently learning section, and contact info. Deployed to the maryem2005/maryem2005 special GitHub repo so it shows at the top of my profile.
 
**Evaluation:** Clean and professional. Much better than having an empty profile. The featured project section directly links to my component work.
 
**What I learned:** Your GitHub profile is often the first thing a recruiter sees. Having a profile README that explains what you build and links to real work makes a much stronger impression than a list of repo names with no context.
 
---
 
## Reflection
 
When I started this project in Week 8, my prompts were broad and generic — "help me build a quiz generator" or "write a README." The outputs were generic too.
 
By Week 11-12, my prompts were specific and technical — I was describing exact field names, node types, error conditions, and expected behaviors. The outputs matched: targeted fixes, accurate expressions, real debugging help.
 
The biggest shift was learning that AI tools work best when you give them the same context a knowledgeable teammate would need. The more I treated Claude and Copilot like a collaborator who needed to understand my actual system, the more useful their responses became.
