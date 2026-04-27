## 2026-04-27 — Re-ran capstone audit for Checkpoint 2

**Context:** Week 9 lab, capstone repo open in VS Code, updated copilot-instructions.md loaded

**Prompt:**
> I need you to act as a capstone project advisor... (full audit prompt)

**Result:** Generated updated gap analysis showing Ingestion complete, AI Core mostly working, Specialist incomplete, Integration partially built

**Evaluation:** Accurate - correctly identified AI Core → Specialist handoff as the main blocker

**What I changed:** Nothing, the analysis matched our actual project state

**What I learned:** Re-running the same audit after a week shows clear progress - Ingestion and AI Core moved from "in progress" to "complete"

---

## 2026-04-27 — Debugged n8n pipeline for Week 8 lab

**Context:** Building Week 8 LLM chain pipeline in n8n, HTTP Request nodes failing with invalid JSON error

**Prompt:**
> My n8n HTTP Request node is failing with "not valid JSON" error when passing Flowise output to the next node. How do I fix it?

**Result:** Switched from "Using JSON" to "Using Fields Below" in the HTTP Request body configuration

**Evaluation:** Worked - all nodes turned green after the fix

**What I changed:** Changed body specification method in HTTP Request1 and HTTP Request2 nodes

**What I learned:** When Flowise returns text with special characters, using field-based body specification instead of raw JSON prevents parsing errors

---

## 2026-04-27 — Generated Flowise LLM chains for security analysis

**Context:** Week 8 lab, building 3 Flowise chains for alert classification, threat analysis, and response recommendations

**Prompt:**
> You are a senior security analyst. Classify the following alert using severity levels CRITICAL, HIGH, MEDIUM, LOW, INFO. Respond with JSON only.

**Result:** Chain correctly classified SSH brute force attempt as MEDIUM/HIGH with confidence score and reasoning

**Evaluation:** Worked well - valid JSON output every time, severity levels were appropriate

**What I changed:** Had to swap prompts manually since Flowise free tier only allows 2 chatflows

**What I learned:** Temperature setting matters - 0.2 gives more consistent JSON output than higher temperatures

---

## 2026-04-27 — Used Copilot to generate checkpoint2-results.md

**Context:** Week 9 lab, needed to document end-to-end test results for Checkpoint 2

**Prompt:**
> Write a checkpoint2-results.md file documenting our end-to-end test. Ingestion working (21 ready records), AI Core working (199 questions), Specialist partial, Integration partial.

**Result:** Generated complete structured results document with component-by-component breakdown, gaps found, and fix plan

**Evaluation:** Very accurate - matched our actual project state and identified the right gaps

**What I changed:** Minor edits to screenshot filenames and specific details

**What I learned:** Giving Copilot the actual numbers and status makes the output much more specific and useful than vague descriptions
