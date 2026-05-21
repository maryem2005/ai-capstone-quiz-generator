# Limitations and Constraints

## 1. Airtable Interface Limitations

Airtable was useful for organizing the backend data, but it was not ideal as a true quiz-taking interface.

Main issues:
- long question text could get visually cut off
- the interface was not as clean as a custom web app
- users could accidentally see technical fields like `record_id`, `status`, or linked record fields
- hiding unnecessary fields required extra interface cleanup
- it was difficult to make the quiz experience feel natural for the user

At one point, users had to manually select their own questions, which was not ideal because the system was intended to automatically generate and deliver quizzes.

We improved the interface as much as possible by hiding backend fields and organizing the visible fields more clearly.

---

## 2. Question Display Constraint

Airtable does not easily support a polished quiz layout where each question appears cleanly with its own answer input.

Because of this, the final quiz-taking process required a workaround:
- users view the questions
- users keep track of their answers
- users submit answers separately through response fields

This worked for the prototype, but a full production version would benefit from a custom frontend.

---

## 3. Answer Choice Formatting

A positive improvement was that Flowise successfully generated multiple-choice answer choices labeled A-D.

This allowed the system to create clearer quiz questions with:
- option A
- option B
- option C
- option D
- correct answer
- explanation

This improved quiz readability and made automated grading more practical.

---

## 4. n8n Execution / Paywall Limitation

The system worked when workflows were manually executed.

However, n8n’s execution limits prevented the system from running fully automatically after publishing.

Because of this, the final version required manual execution for some workflows.

This was a platform limitation rather than a logic failure in the project.

[Insert screenshot of n8n execution limit/paywall issue here]

---

## 5. Flowise Endpoint Sensitivity

A major limitation was that Flowise integrations were sensitive to changes.

Every time the Flowise chatflow was changed, the HTTP request link in n8n sometimes had to be updated.

If the old link stayed in the HTTP Request node, the workflow could break even if the Airtable fields and prompt were correct.

This made debugging harder because the issue was not always obvious.

---

## 6. Prompt Variable Mapping Issues

The Flowise prompt required very specific input variables.

During testing, n8n sometimes sent missing or undefined values into Flowise.

Common issues included:
- missing `notes`
- wrong node references
- incorrect JSON paths
- clean text not being passed correctly
- topic or record ID not appearing in the expected node

This caused errors where Flowise could not process the request because required prompt values were missing.

---

## 7. Metadata Propagation Problems

A recurring challenge was getting metadata to carry across every table.

Fields such as:
- document
- topic
- quiz
- response
- performance
- user_id
- attempt_id
- source
- created_at
- submitted_at
- graded_at
- generation_id
- confidence_score

did not always transfer correctly between workflows.

This made it harder to trace one record cleanly through the full system.

---

## 8. Airtable Linked Record Complexity

Linked records were useful, but they also made the workflow harder to manage.

The system needed links between:
- Documents and Questions
- Questions and Quizzes
- Quizzes and Responses
- Responses and Performance

If one link was missing, the dashboard and grading logic became harder to trace.

---

## 9. Original Workflow Was Too Coupled

The original n8n workflow design became too connected across multiple components.

This created issues because:
- one person’s workflow depended on another person’s workflow
- errors were hard to isolate
- fixing one section sometimes caused another issue
- debugging took longer than expected

This is why the final system was redesigned into three separate workflows.

---

## 10. Manual Workarounds Were Needed

Because of Airtable and n8n limitations, some parts of the system still required manual workarounds.

Examples:
- manually executing workflows
- manually checking if records populated correctly
- manually hiding technical Airtable fields
- manually testing whether Flowise outputs matched Airtable fields

These workarounds were acceptable for the prototype but would need to be automated in a production version.

---

## 11. Dashboard Was Limited by Available Data

The dashboard could only show insights if the correct fields were populated.

When fields like weak topics, score percent, confidence score, or completed timestamps were missing, dashboard reporting became limited.

This showed how important clean metadata is for analytics.

---

## 12. No Custom Frontend

The biggest long-term limitation is that the project used Airtable as both database and interface.

Airtable worked for a prototype, but a custom frontend would provide a much better experience.

A future version should use a dedicated web interface where users can:
- upload documents
- take quizzes directly
- select answers beside each question
- submit responses naturally
- view results in a clean dashboard
