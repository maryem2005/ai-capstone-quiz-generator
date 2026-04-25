# Prompt Log — Maryem

**Project:** AI Quiz Generator
**Team:** MarialsCoding
**My Component:** AI Core / Question Generator
**AI Tools Used:** GitHub Copilot

---

## 2026-04-24 — Capstone Audit Interview

**Context:** Had copilot-instructions.md open in VS Code with full project context loaded. Running Week 8 lab capstone audit before Checkpoint 2.

**Prompt:**
> I need you to act as a capstone project advisor for a university AI integration course. Interview me about my project's current state and produce a structured gap analysis.

**Result:** Copilot interviewed me with 10 questions one at a time and produced a full Checkpoint 2 Readiness Assessment. Status: AT RISK. Identified 4 critical gaps including my unbuilt Step 7 workflow and Noeleen's unstarted component.

**Evaluation:** Accurate and honest. The gaps it identified are real — especially Step 7 being unbuilt and no end-to-end testing done yet.

**What I changed:** Nothing — the report was accurate as generated.

**What I learned:** Having copilot-instructions.md loaded made Copilot extremely specific to our actual project — it referenced real field names like document_id, options_json, and raw_text instead of giving generic advice.

---

## 2026-04-24 — AI Core Component README Generation

**Context:** copilot-instructions.md loaded in VS Code. Asked Copilot to generate a README for my component.

**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Question Generator component. Include what it does, how it connects to other components, setup instructions, how to test it, and known limitations.

**Result:** Generated a complete README with actual field names, table names, setup instructions for Groq and n8n, test steps, and known limitations including the document_id linked record risk and options_json parsing issue.

**Evaluation:** Excellent. Referenced our actual schema and tools correctly. Known limitations section was especially useful — identified real risks we need to address before Checkpoint 2.

**What I changed:** Added it to the bottom of my existing README.md to preserve Week 4 and Week 5 history.

**What I learned:** The more specific your instructions file, the more useful the generated artifact. Generic prompts give generic READMEs — project context gives project-specific ones.

## Entry 6
**Date:** April 24, 2026
**Tool:** Claude
**Prompt:** "Help me add error handling to my n8n workflow for when the Groq API fails"
**Output:** Added If1 node to detect API errors and Update record1 node to write status = "error" to Airtable Documents table
**Evaluation:** Worked well — workflow now catches API failures gracefully instead of crashing

## Entry 7
**Date:** April 24, 2026
**Tool:** Claude
**Prompt:** "Help me set up confidence-based routing using question difficulty in my IF node"
**Output:** Updated IF node to route hard questions to reviewed path and easy/medium to generated path
**Evaluation:** Successfully routes questions based on difficulty level

## Entry 8
**Date:** April 24, 2026
**Tool:** Claude
**Prompt:** "Help me create Airtable dashboard views for error monitoring and pipeline status"
**Output:** Created Error Monitor view filtered by status=error and Pipeline Status view grouped by status
**Evaluation:** Dashboard now shows system health at a glance
