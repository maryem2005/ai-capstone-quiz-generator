# Week 10 Lab Submission
**Name:** Maryem Elgebaly
**Date:** April 24, 2026
**GitHub Repo:** https://github.com/maryem2005/ai-capstone-quiz-generator

## Part 1: Error Handling

**What error I handled:** API failure — when the Groq HTTP Request node fails or returns an error.

**How I handled it:** I enabled "Continue on Fail" on the HTTP Request node and added a new IF node (If1) after it that checks if `$json.error` is not empty. If true (API failed), the workflow routes to a new Update record1 node that sets `status = "error"` in the Documents table so the record is visible and retryable. If false (success), the workflow continues normally to the Code in JavaScript node.

**Screenshots:** error-handling-workflow.png, error-handling-test-record.png

---

## Part 2: Confidence-Based Routing

**Routing logic:** I used question difficulty as my confidence equivalent. After the AI generates questions, the IF node checks if `$json.difficulty` is equal to "hard". Hard questions route to the reviewed path (needs human verification) while easy and medium questions route to the generated path (auto-approved for the specialist).

**Threshold justification:** I chose difficulty level as my threshold because hard questions are more likely to contain errors or require subject matter expertise to verify, while easy and medium questions are straightforward enough to pass through automatically.

**Screenshots:** confidence-routing-workflow.png, confidence-routing-records.png

---

## Part 3: Dashboard Views

**Error Monitor view:** Filtered to show only Documents with status = "error" — this is for the whole team to quickly see which documents failed processing and need to be reprocessed.

**Pipeline Status view:** Groups all 25 documents by status (ready vs error) — this is for the team to see the overall health of the pipeline at a glance.

**Screenshots:** dashboard-error-monitor.png, dashboard-pipeline-status.png

---

## Part 4: Prompt Log
See prompt-log-maryem.md — updated with entries 6, 7, and 8 from this week's work.
