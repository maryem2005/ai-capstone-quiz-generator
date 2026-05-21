# Prompt Log — Maria

This log documents examples of AI-assisted development support used throughout the AI Quiz Generator project. GitHub Copilot was used as a coding/debugging assistant to help identify integration issues, suggest workflow improvements, generate test data, and support implementation decisions.

---

## Entry 1 — Debugging Flowise HTTP Request Failure

**Date:** May 2026

### Prompt
My n8n HTTP Request node is failing when sending data to Flowise. The error says:

`Please provide Prompt Values for: $json.notes`

The Airtable record clearly contains text, but the HTTP Request body says the value is undefined. How can I debug this?

---

### Copilot Response (Expected)

The error indicates that Flowise expects a variable named `notes`, but the value being passed from n8n is missing or incorrectly mapped.

Recommended debugging steps:

1. Check the exact variable name expected in your Flowise prompt template.
2. Confirm the HTTP Request JSON body matches that variable exactly.
3. Verify the correct upstream node is being referenced.

Example:

Incorrect:
```json
{
  "notes": {{$json.notes}}
}
```

Correct if the value is coming from a previous Airtable node:
```json
{
  "notes": "{{$node['Search records'].json['fields']['clean_text']}}"
}
```

Also verify:
- the Airtable field actually contains data
- the node executing immediately before HTTP Request contains the expected value
- no renamed fields broke the reference chain

---

### Outcome

This helped identify that the wrong node reference was being used inside the HTTP Request body, causing Flowise to receive undefined prompt values.

---

# Entry 2 — Flowise Endpoint / Link Expiration Issue

**Date:** May 2026

### Prompt

My Flowise workflow was working earlier, but after editing the chatflow, my n8n HTTP Request stopped working even though the payload looks correct. Could the Flowise endpoint have changed?

---

### Copilot Response (Expected)

Yes.

Flowise deployments may generate a different endpoint URL if the chatflow is redeployed or recreated.

Checklist:

- open Flowise
- confirm the currently deployed prediction endpoint
- compare it against the URL inside your n8n HTTP Request node

Example:

Old:
```bash
https://flowise.example.com/api/v1/prediction/OLD_ID
```

New:
```bash
https://flowise.example.com/api/v1/prediction/NEW_ID
```

If these do not match, n8n will fail even if the request body is valid.

Recommended fix:
Update the HTTP Request URL inside n8n to the current deployed Flowise endpoint.

---

### Outcome

This reflected one of the major hidden integration issues discovered during debugging.

---

# Entry 3 — Fixing option_a / option_b / option_c / option_d Mapping

**Date:** May 2026

### Prompt

Flowise is generating quiz questions, but Airtable’s Questions table is not consistently populating option_a, option_b, option_c, and option_d correctly.

How should the output schema be structured?

---

### Copilot Response (Expected)

Your AI output should be forced into a predictable JSON schema.

Example expected structure:

```json
{
  "question_text": "What was the main cause of World War I?",
  "question_type": "multiple_choice",
  "option_a": "Assassination of Archduke Franz Ferdinand",
  "option_b": "The Cold War",
  "option_c": "Industrialization",
  "option_d": "The Great Depression",
  "correct_answer": "A",
  "explanation": "The assassination triggered alliances into war.",
  "difficulty": "medium"
}
```

Recommendations:

- enforce strict JSON output in the Flowise prompt
- validate that every MCQ includes all four options
- reject malformed outputs
- ensure Airtable field names exactly match:
  - option_a
  - option_b
  - option_c
  - option_d

---

### Outcome

This supported schema consistency for question generation.

---

# Entry 4 — Generating Test Data for Integration Testing

**Date:** May 2026

### Prompt

Generate realistic test data for my AI Quiz Generator pipeline.

I need uploaded study material examples that include normal cases, edge cases, and failure cases.

---

### Copilot Response (Expected)

Recommended integration test dataset:

Normal Cases:
- World History lecture notes
- Biology textbook chapter
- Psychology study guide
- Computer science lecture slides

Edge Cases:
- very short document
- empty upload
- scanned PDF with OCR noise
- malformed file
- oversized document
- duplicate upload

Suggested Airtable test fields:

```csv
title,course,topic,status
World History Notes,History,World Wars,uploaded
Biology Chapter 5,Biology,Cell Structure,uploaded
Psychology Review,Psychology,Memory,uploaded
Blank File,Unknown,None,error
```

---

### Outcome

Test records were used to validate ingestion and downstream workflows.

---

# Entry 5 — Lecture Notes / AI Test Content Generation

**Date:** May 2026

### Prompt

Generate 5 pages of realistic educational lecture notes I can use to test my AI quiz generation system.

Topics:
- World War I
- World War II
- Earth Science

---

### Copilot Response (Expected)

Generated long-form educational lecture content including:

- historical summaries
- major events
- key figures
- timelines
- concept explanations

This content was suitable for:

- document upload testing
- chunking validation
- AI question generation
- grading pipeline testing

---

### Outcome

AI-generated lecture notes were used as realistic test inputs.

---

# Entry 6 — Redesigning the Response Submission Architecture

**Date:** May 2026

### Prompt

My original design stores one response record per question.

But Airtable’s interface makes that awkward for users.

Would it be better to submit 10 answers in one response record instead?

---

### Copilot Response (Expected)

Yes.

Current normalized model:

- one response per question

Pros:
- cleaner relational database design

Cons:
- poor Airtable UX
- excessive record creation
- harder grading logic
- fragmented submission experience

Alternative design:

Store one full quiz attempt:

```text
submitted_1
submitted_2
submitted_3
submitted_4
submitted_5
submitted_6
submitted_7
submitted_8
submitted_9
submitted_10
```

Updated grading workflow:

1. trigger on one response submission
2. retrieve selected quiz
3. retrieve linked questions
4. map answers
5. compare against correct answers
6. compute score
7. write performance record

---

### Outcome

This became the final workflow architecture because it was significantly more user-friendly.

---

# Entry 7 — Re-Auditing the Entire Workflow

**Date:** May 2026

### Prompt

Audit this AI Quiz Generator architecture.

Current stack:
- Airtable
- n8n
- Flowise

Look for architectural weaknesses, workflow coupling, metadata issues, and automation gaps.

---

### Copilot Response (Expected)

Audit findings:

Critical risks:
- tightly coupled workflows
- missing automation triggers
- incomplete metadata propagation
- Airtable frontend limitations
- brittle Flowise integration
- dependency on manual execution

Recommendations:
- split workflows into modular components
- reduce interdependencies
- simplify response schema
- improve metadata consistency
- separate frontend concerns from database logic

---

### Outcome

This aligned closely with the eventual decision to rebuild the architecture.
