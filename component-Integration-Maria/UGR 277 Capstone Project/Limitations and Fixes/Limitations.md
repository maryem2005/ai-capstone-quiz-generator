# Limitations and Constraints

## 1. Airtable Interface Limitations

One of the most significant technical limitations came from Airtable itself being used as both the backend database and the user-facing quiz interface.

While Airtable worked well for storing structured data and managing linked records, it was not designed to function as a true interactive quiz platform.

This created several architectural constraints that directly forced redesign decisions.

#### Inability to Present Questions and Answer Inputs Naturally

The biggest issue was that Airtable could not cleanly support the type of quiz-taking experience we originally intended.

The ideal user experience would have looked like:

- Question 1 displayed
- Answer input directly beneath it
- Question 2 displayed
- Answer input directly beneath it
- repeated dynamically for however many AI-generated questions existed

However, Airtable’s interface builder did not support this type of dynamic side-by-side quiz interaction in a practical way.

Specific problems included:

- generated questions and answer submission fields could not be naturally paired together
- questions and response inputs could not be dynamically rendered as a traditional quiz layout
- question display and answer collection had to be separated rather than integrated
- Airtable forms are record-oriented, not interaction-oriented

This meant the system could not function like a true quiz application where users simply move through questions and answer naturally.

Instead, users had to:

1. open the quiz view
2. read the displayed questions
3. manually keep track of their answers externally or mentally
4. navigate to the answer submission form
5. manually enter responses into generic fields like:
   - `submitted_1`
   - `submitted_2`
   - `submitted_3`
   - etc.

This was one of the most significant UX compromises in the final implementation.

---

#### Original One-Response-Per-Question Schema Became Impractical

The original database design followed a more normalized structure:

- one response record per question
- each response linked individually to:
  - a quiz
  - a question
  - a performance record

From a database design perspective, this was clean.

However, Airtable’s interface made this model impractical.

In theory, this would require:

- dynamically generating a response form for each individual question
- presenting one question with one answer field
- submitting each answer separately
- linking each response record correctly
- then aggregating all responses later for grading

Airtable could not support this elegantly.

Problems included:

- excessive user clicking
- fragmented quiz experience
- awkward navigation between questions
- too many records created per attempt
- more difficult grading logic
- heavier workflow automation requirements
- complicated linked-record management

Although structurally sound, the design created a poor real-world user experience.

---

#### Final Schema Was Redesigned Around Platform Constraints

Because of Airtable’s limitations, the schema had to be redesigned around what the platform could realistically support.

Instead of:

**one response record per question**

the final implementation used:

**one response record per quiz attempt**

This meant a single response record stored multiple answers:

- `submitted_1`
- `submitted_2`
- `submitted_3`
- `submitted_4`
- `submitted_5`
- `submitted_6`
- `submitted_7`
- `submitted_8`
- `submitted_9`
- `submitted_10`

The grading workflow then became:

1. user submits one completed quiz attempt
2. n8n triggers once
3. selected quiz is retrieved
4. linked questions are pulled
5. answers are mapped:
   - `submitted_1 → Question 1`
   - `submitted_2 → Question 2`
   - etc.
6. grading occurs in batch
7. performance analytics are generated

This was less normalized from a pure database design perspective, but significantly more practical given Airtable’s interface limitations.

---

#### Airtable Interface Display Issues

Even after redesigning the schema, presentation limitations remained.

Problems included:

- long AI-generated questions being visually cut off
- limited control over formatting and spacing
- inability to create polished quiz UI layouts
- difficulty making the interface feel intuitive
- backend fields needing manual hiding

A significant amount of interface cleanup was required just to prevent users from seeing technical implementation details such as:

- `record_id`
- `status`
- linked record references
- internal metadata fields

Without this cleanup, the interface looked more like a database admin panel than a student-facing educational product.

---

#### Existing Airtable Quiz Implementations Were Not Useful for Dynamic AI Workflows

Before redesigning the interface, I researched how others had built quiz systems in Airtable through YouTube tutorials and documentation.

A major limitation quickly became clear:

most examples assumed static quiz content.

Typical implementations required users to:

- manually choose questions
- select from fixed question banks
- take the same predefined quiz each time

This did not align with our system.

Our quizzes were fundamentally dynamic because:

- AI generated new questions every run
- question sets changed depending on uploaded material
- quizzes were not static assets

As a result, existing Airtable tutorials were not directly applicable.

This meant much of the interface architecture had to be designed through experimentation rather than established implementation patterns.

---

#### Airtable Was Effective as a Prototype Tool, Not a Production Frontend

Airtable was extremely useful for rapid prototyping because it allowed:

- quick schema creation
- linked record relationships
- form creation
- simple interface building
- lightweight dashboard functionality

However, as a production quiz platform, it introduced serious UX and interaction limitations.

A custom frontend would allow:

- dynamic question rendering
- answer inputs beside each question
- cleaner quiz navigation
- better formatting
- stronger validation
- improved usability
- more natural interaction flow

The final implementation succeeded as a functional prototype, but the interface limitations strongly influenced architectural decisions throughout the project.

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
