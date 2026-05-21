# Airtable Schema, Views, Team Coordination, and Integration Notes

## Original Airtable Schema Design

To support the full AI Quiz Generator pipeline, the Airtable database was originally designed with five separate tables:

- Documents
- Questions
- Quizzes
- Responses
- Performance

Each table was built around four shared system fields:

- `record_id`
- `created_at`
- `source`
- `status`

These fields were included to keep records consistent, traceable, and easier to debug across the full workflow.
# Design Decision: Confidence Score Interpretation

One implementation decision our team had to make involved how to represent the system’s confidence score.

The original project requirements referenced including a confidence score, but did not explicitly define:

- what the confidence score should measure
- how it should be calculated
- whether it should represent model certainty, grading certainty, or another evaluation metric
- whether it should be expressed as a probability (0–1) or a percentage (0–100)

Because this was not formally specified, our team made an implementation judgment based on what made the most practical sense for our workflow.

We defined the confidence score as:

> **the AI grading system’s level of certainty in the accuracy of its grading decision for a student’s submitted quiz responses.**

Rather than representing this as a decimal probability between 0 and 1, we intentionally chose a **0–100 percentage scale**.

Example interpretation:

- **95** → the grading system is highly confident in its evaluation
- **72** → moderate confidence in grading accuracy
- **40** → lower confidence, suggesting the response may require closer review

We chose this approach because the grading output is tied directly to student performance assessment, and a percentage-based scale was significantly more intuitive and easier for end users to interpret than raw probability values.

For example:

- `0.87 confidence` may be technically standard in machine learning contexts
- `87% confidence` is immediately understandable to most users

This was ultimately a usability-driven design decision rather than a mathematically mandated implementation.

## Reflection

This also highlighted an important systems design lesson:

Project requirements sometimes define *what* must exist without fully defining *how* it should behave.

In those situations, implementation teams must make reasonable architectural decisions that balance:

- technical correctness
- usability
- interpretability
- implementation practicality

Our confidence score design reflects that type of engineering judgment.

---

# Documents Table

The Documents table was designed as the entry point of the system. It stores uploaded study files and tracks how each document moves through the processing pipeline, from raw upload to AI-ready chunked text.

## Core System Fields

| Field | Purpose |
|---|---|
| `record_id` | Unique auto-generated ID for each document |
| `created_at` | Timestamp of when the document was added |
| `status` | Tracks document stage: uploaded, extracting, cleaned, chunked, ready, error |
| `source` | Indicates where the document came from, such as upload, URL, or API |

## File and Metadata Fields

| Field | Purpose |
|---|---|
| `file_upload` | Stores the uploaded file |
| `title` | Human-readable document title |
| `file_name` | Original filename from upload |

## Organization Fields

| Field | Purpose |
|---|---|
| `course` | Course connected to the document |
| `topic` | Topic covered in the document |
| `user_id` | Identifies who uploaded the document |

## Processing Fields

| Field | Purpose |
|---|---|
| `raw_text` | Extracted text from the uploaded document |
| `clean_text` | Cleaned version of the extracted text |
| `chunked_sections` | Text split into smaller sections for AI generation |
| `chunk_count` | Number of chunks created |
| `processed_text_ref` | Optional workflow or processing reference |

## Linking Field

| Field | Purpose |
|---|---|
| `questions` | Links each document to generated questions |

---

# Questions Table

The Questions table stores every AI-generated quiz question. It connects source documents to quizzes and user responses.

## Core System Fields

| Field | Purpose |
|---|---|
| `record_id` | Unique auto-generated ID for each question |
| `created_at` | Timestamp of when the question was created |
| `status` | Tracks lifecycle: unprocessed, generating, generated, reviewed, approved, error |
| `source` | Indicates whether the question came from AI, manual entry, or API |

## Linking Fields

| Field | Purpose |
|---|---|
| `document` | Links question back to its source document |
| `quiz` | Links question to a quiz |
| `responses` | Links question to submitted responses |

## Question Content Fields

| Field | Purpose |
|---|---|
| `question_text` | The question being asked |
| `question_type` | Multiple choice, true/false, short answer, or fill-in-the-blank |
| `option_a` | First answer choice |
| `option_b` | Second answer choice |
| `option_c` | Third answer choice |
| `option_d` | Fourth answer choice |
| `correct_answer` | Correct answer used for grading |
| `explanation` | Explanation shown to the user |

## Metadata Fields

| Field | Purpose |
|---|---|
| `difficulty` | Easy, medium, or hard |
| `topic` | Topic tested by the question |
| `question_number` | Order of the question within a quiz |
| `ai_generated` | Indicates whether the question was created by AI |
| `generation_id` | Groups questions from the same AI generation run |
| `generated_at` | Timestamp of AI generation |
| `is_used` | Tracks whether the question has already been used |

---

# Quizzes Table

The Quizzes table represents a complete quiz session. It groups questions together and connects to responses and performance records.

## Core System Fields

| Field | Purpose |
|---|---|
| `record_id` | Unique auto-generated quiz ID |
| `created_at` | Timestamp of quiz creation |
| `status` | Generated, in-progress, completed, or error |
| `source` | Indicates how the quiz was created |

## Quiz Structure Fields

| Field | Purpose |
|---|---|
| `quiz_title` | Display name of the quiz |
| `topic` | Subject or topic covered |
| `difficulty` | Overall quiz difficulty |
| `quiz_type` | Multiple choice, mixed, timed, etc. |
| `questions` | Linked questions included in the quiz |
| `question_count` | Number of questions in the quiz |

## User and Tracking Fields

| Field | Purpose |
|---|---|
| `user_id` | Identifies the user |
| `document` | Links quiz to source document |
| `time_limit_minutes` | Optional timed quiz setting |
| `assigned_at` | When quiz became available |
| `completed_at` | When quiz was finished |
| `generation_id` | Connects quiz to AI generation batch |
| `is_active` | Indicates whether quiz is active |

## Results Fields

| Field | Purpose |
|---|---|
| `responses` | Links submitted responses |
| `performance` | Links final performance record |

---

# Responses Table

The Responses table was originally designed to store each individual answer submitted by a user.

## Core System Fields

| Field | Purpose |
|---|---|
| `record_id` | Unique response ID |
| `created_at` | Timestamp of response creation |
| `status` | Submitted, graded, or error |
| `source` | Submission source |

## Linking Fields

| Field | Purpose |
|---|---|
| `quiz` | Links response to quiz |
| `question` | Links response to question |
| `performance` | Links response to final performance summary |

## User and Attempt Fields

| Field | Purpose |
|---|---|
| `user_id` | Identifies user |
| `attempt_id` | Groups all responses from the same attempt |

## Answer and Grading Fields

| Field | Purpose |
|---|---|
| `submitted_answer` | User’s submitted answer |
| `correct_answer` | Correct answer used for comparison |
| `is_correct` | Indicates whether answer was correct |
| `score_awarded` | Points earned |
| `feedback` | Feedback shown to the user |
| `missed_topic` | Topic missed if answer was incorrect |
| `response_type` | Type of response |
| `submitted_at` | Submission timestamp |
| `graded_at` | Grading timestamp |

---

# Performance Table

The Performance table stores final results for a quiz attempt.

## Core System Fields

| Field | Purpose |
|---|---|
| `record_id` | Unique performance record ID |
| `created_at` | Timestamp of record creation |
| `status` | Calculating, completed, or error |
| `source` | Source of performance calculation |

## Linking Fields

| Field | Purpose |
|---|---|
| `quiz` | Links to quiz |
| `responses` | Links to included responses |

## User and Attempt Fields

| Field | Purpose |
|---|---|
| `user_id` | Identifies user |
| `attempt_id` | Tracks multiple attempts |

## Scoring Fields

| Field | Purpose |
|---|---|
| `total_questions` | Total number of questions |
| `correct_count` | Number correct |
| `incorrect_count` | Number incorrect |
| `score_raw` | Raw score |
| `score_percent` | Percentage score |

## Learning Insight Fields

| Field | Purpose |
|---|---|
| `weak_topics` | Topics needing improvement |
| `strong_topics` | Topics performed well |
| `overall_feedback` | Final feedback summary |

## Timing and Outcome Fields

| Field | Purpose |
|---|---|
| `started_at` | Quiz start time |
| `completed_at` | Quiz completion time |
| `time_spent_minutes` | Total time spent |
| `passed` | Pass/fail indicator |

---

# Original Airtable Views and Data Initialization

After finalizing the original schema, I created the first operational Airtable views so the team could interact with the data more efficiently.

The goal was to build:

- filtered views for each component
- views for debugging
- a Kanban board for workflow tracking
- sample records for testing

---

## Documents Table Views

| View | Purpose |
|---|---|
| Ready for Processing | Filtered by `status = uploaded` |
| Errors | Filtered by `status = error` |
| By Course | Grouped by course, such as Psychology, History, and Biology |

---

## Questions Table Views

The Questions table already had sample generated questions. I manually added additional realistic sample records across Psychology, History, and Biology.

These included:

- multiple choice questions
- true/false questions
- short answer questions
- difficulty levels
- topics
- explanations
- generated and approved statuses

Views created:

| View | Purpose |
|---|---|
| Approved Questions | Shows questions ready for quizzes |
| Needs Review | Shows generated questions awaiting review |
| Question Workflow Kanban | Groups questions by status |

---

## Quizzes Table Views

The Quizzes table was initialized with a sample quiz record.

Views created:

| View | Purpose |
|---|---|
| Active Quizzes | Shows active quizzes |
| Completed Quizzes | Shows quizzes with completed timestamps |

---

## Responses Table Views

The Responses table was initialized with a sample response record.

Views created:

| View | Purpose |
|---|---|
| Ungraded Responses | Shows submitted responses awaiting grading |
| Graded Responses | Shows responses already scored |

---

## Performance Table Views

The Performance table was initialized with a sample performance record.

View created:

| View | Purpose |
|---|---|
| Completed Results | Shows finalized quiz performance summaries |

---

## Initial Record Count

Approximate records after initialization:

| Table | Approximate Records |
|---|---|
| Documents | 25 |
| Questions | 12+ |
| Quizzes | 1+ |
| Responses | 1+ |
| Performance | 1+ |

Total: 40+ records

This exceeded the required 30+ test records and created a realistic development environment for the team.

---

# Team Coordination and Integration Leadership

As the Integrator, my role was to keep the team aligned and ensure each component moved toward a connected final system.

I created and maintained a master checklist that assigned:

- owner
- task
- expected output
- dependency order
- definition of done

This checklist served as the main project coordination document.

---

## Original Team Checklist

| Owner | Responsibility |
|---|---|
| Maria | Finalize Airtable schema |
| Ryan | Create initial test dataset |
| Maria | Populate 30+ records and build Airtable views |
| Ryan | Build n8n ingestion workflow |
| Maryem | Test candidate models |
| Noeleen | Write component README and research Flowise/Groq |
| Maryem | Finalize model selection and HTTP Request node |
| Noeleen | Build paper prototype for quiz delivery |
| Ryan | Demo ingestion workflow |
| Maryem | Build Flowise Question Generator chain |
| Noeleen | Build Quiz Delivery and Scoring skeleton |
| Maryem | Load knowledge base and vector store |
| Maria | Run Checkpoint 1 |
| Noeleen | Complete core Quiz Delivery and Scoring |
| Maria | Run Checkpoint 2 |
| Ryan | Add error handling |
| Maryem | Add confidence-based routing |
| Noeleen | Add edge case support |
| Maria | Build production Airtable dashboard |
| Maria | Run Checkpoint 3 and publish GitHub materials |
| Full Team | Final integration test and demo rehearsal |

---

# Integration Challenge

The original plan depended on each component being completed in order.

Ryan’s ingestion workflow had to be stable before Maryem’s AI generation workflow could be tested. Maryem’s generated questions then had to be stable before Noeleen’s quiz delivery and scoring component could work.

The project became blocked when the quiz delivery and scoring component was not completed as expected.

This created a major dependency issue because Checkpoint 2 relied on a functioning end-to-end path.

---

# Revised Plan and Updated Checklist

When the original plan became blocked, I created a revised plan to keep the project moving.

This involved:

- redistributing missing responsibilities
- simplifying the remaining build path
- focusing on a working demo
- prioritizing core functionality over optional features

## Revised Checklist

| Owner | Responsibility |
|---|---|
| Maria | Run Checkpoint 1 and document field mismatches |
| Maryem | Complete Quiz Delivery and Scoring skeleton |
| Maryem | Complete core scoring functionality |
| Ryan | Add error handling |
| Maryem | Add confidence-based routing |
| Ryan | Add edge case handling |
| Maria | Run Checkpoint 2 and build dashboard |
| Maria | Run Checkpoint 3 and publish GitHub documentation |
| Full Team | Final integration test, demo rehearsal, README cleanup |

---

# New Airtable Redesign

After additional testing, it became clear that the original schema was too complex for the final working version.

The biggest issue was that the original Responses design stored one response per question. While this was theoretically clean, it created too much workflow complexity inside Airtable and n8n.

The redesigned Airtable base simplified the system while keeping the same core architecture.

---

## Key Redesign Decisions

### 1. Simplified Response Submission

Instead of creating one response record per question, the redesigned system stores one full quiz attempt in a single Responses record.

The new Responses table includes:

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

This made it easier to:

- collect answers through Airtable
- pass submissions into n8n
- grade multiple answers together
- connect the response to one performance record

---

### 2. Three Separate n8n Workflows

The rebuilt system uses three separate workflows:

1. Document ingestion and AI question generation
2. Quiz delivery and answer submission
3. Grading and performance analytics

This replaced the earlier tightly coupled structure.

Benefits:

- easier debugging
- clearer responsibility separation
- fewer cascading errors
- easier demo preparation

---

### 3. Cleaner Field Naming

The redesigned schema reduced confusing or unnecessary fields.

Fields that were not actively used were either removed or left in place only if deleting them risked breaking the workflow.

Some fields, such as `completed_at` in Performance, were kept even though they were not fully used in the final version. These were retained to avoid damaging working automations close to the deadline.

---

### 4. Stronger Traceability

The redesigned schema preserved important links between:

- documents
- generated questions
- quizzes
- responses
- performance records

This helped support debugging and final demo explanation.

---

# New Project Workflow Kanban

A new Project Workflow Kanban view was added to the shared Airtable base.

This view groups records by processing status and gives a visual representation of the AI quiz generation pipeline.

The Kanban supports tracking across stages such as:

- uploaded
- extracting
- cleaned
- chunked
- ready
- generated
- submitted
- graded
- completed
- error

This made it easier to monitor:

- document progression
- AI processing
- quiz generation
- grading status
- failed or stuck records

---

# Struggles and Debugging Challenges

The project involved several major technical and coordination challenges.

## Workflow Trigger Problems

The system did not originally run automatically after document upload.

Instead, records often required:

- manual status updates
- manual workflow execution
- manual triggering inside n8n

This was a major issue because the project was supposed to behave like an automated pipeline.

---

## Flowise HTTP Request Issues

One of the most difficult problems was getting n8n to correctly send values into Flowise.

Issues included:

- missing prompt variables
- undefined JSON values
- incorrect node references
- incorrect field names
- Flowise rejecting requests because required prompt values were missing

This required repeated testing of the HTTP Request body and careful checking of which n8n node actually contained the correct `clean_text`, `chunked_sections`, `topic`, and record ID values.

---

## Flowise Endpoint Changes

A major lesson was that when a Flowise chatflow is changed, the link used in the n8n HTTP Request may also need to be updated.

If the Flowise endpoint changes but the n8n HTTP Request still uses the old URL, the integration breaks even if the prompt and Airtable fields are correct.

This became an important debugging discovery.

---

## Airtable Interface Limitations

Airtable was useful for creating a lightweight interface, but it had limitations.

The biggest issue was quiz-taking.

Airtable could show generated questions and collect submitted answers, but it could not easily behave like a custom quiz website where each question appears directly next to its own answer input in a dynamic form.

Because of this, the final workaround was:

1. user views the quiz questions
2. user writes or tracks their answers
3. user submits answers in fields such as `submitted_1` through `submitted_10`
4. n8n grades the attempt

This was not a failure of the logic. It was a platform limitation.

---

## Metadata Propagation Problems

A recurring issue was missing metadata.

Fields such as:

- `document`
- `topic`
- `user_id`
- `generation_id`
- `attempt_id`
- `submitted_at`
- `graded_at`
- `source`
- `confidence_score`

were not always passed correctly between workflows.

This made it harder to trace records across the pipeline.

---

## Overly Complex Original Design

The original structure became difficult to maintain because too many parts depended on one another.

Problems included:

- extra tables
- unclear field ownership
- repeated fields
- inconsistent naming
- tightly linked workflows
- manual workarounds

The rebuild simplified the project and made the final system easier to explain and demonstrate.

---

## Execution Limit Issue

The final rebuilt workflows worked when manually executed.

However, when attempting to publish and automate the workflow fully, n8n reported an execution limit issue.

Because of that, the final version requires manual execution.

This is a platform/environment limitation, not a failure of the project design.

---

# Final Working System

The final version supports the main project goal:

1. user uploads a document
2. document is processed
3. text is cleaned and chunked
4. Flowise generates AI quiz questions
5. questions are stored in Airtable
6. quiz can be taken by the user
7. answers are submitted
8. n8n grades the answers
9. performance results are stored
10. feedback and weak topic information are generated

---

# Final Reflection

The biggest lesson from this project was that functional pieces are not enough by themselves. A system only works when the components are connected clearly, data moves reliably, and each workflow has a defined responsibility.

The project started with a more complex design, but testing revealed several issues with automation, metadata, triggers, and user interaction. Through debugging and redesign, the final system became more modular, easier to trace, and better suited for a working demonstration.

The final implementation is not perfect, but it demonstrates the full AI-assisted learning workflow and documents the technical decisions, limitations, and lessons learned throughout development.
