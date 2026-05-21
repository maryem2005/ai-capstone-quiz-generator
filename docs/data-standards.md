Got you — the issue is I wrapped parts in code fences inside the markdown, which GitHub renders as visible blocks instead of normal markdown structure. Here’s the **clean copy-paste-ready `data-standards.md`** exactly as it should go into GitHub:

# Data Standards

## Naming Conventions

All Airtable fields follow these standards:

* Use `snake_case`
* Use lowercase status values
* Use `_at` suffix for date/time fields
* Use descriptive linked-record field names
* No spaces or capital letters in field names

Examples:

* `created_at`
* `quiz_title`
* `score_percent`
* `confidence_score`

---

## Universal Fields

Each Airtable table includes these shared fields:

| Field        | Purpose                               |
| ------------ | ------------------------------------- |
| `record_id`  | Unique auto-generated identifier      |
| `created_at` | Timestamp when the record was created |
| `status`     | Current workflow stage                |
| `source`     | Origin of the record                  |

---

## Core Tables

This project uses five primary Airtable tables:

* Documents
* Questions
* Quizzes
* Responses
* Performance

---

## Documents Table

Stores uploaded study materials and tracks document processing.

| Field              | Purpose                        |
| ------------------ | ------------------------------ |
| `record_id`        | Unique document identifier     |
| `created_at`       | Creation timestamp             |
| `status`           | Processing lifecycle stage     |
| `source`           | Record origin                  |
| `file_upload`      | Uploaded file attachment       |
| `course`           | Course name                    |
| `topic`            | Topic covered                  |
| `raw_text`         | Original extracted text        |
| `clean_text`       | Processed text                 |
| `chunked_sections` | AI-ready text chunks           |
| `chunk_count`      | Number of chunks created       |
| `filename`         | Original uploaded file name    |
| `questions`        | Linked generated questions     |
| `created_by`       | User who uploaded the document |

---

## Questions Table

Stores AI-generated quiz questions.

| Field             | Purpose                             |
| ----------------- | ----------------------------------- |
| `record_id`       | Unique question identifier          |
| `created_at`      | Creation timestamp                  |
| `status`          | Question lifecycle stage            |
| `source`          | Origin of the question              |
| `document`        | Linked source document              |
| `question_text`   | Question content                    |
| `question_type`   | MCQ, true/false, short answer, etc. |
| `option_a`        | Answer choice A                     |
| `option_b`        | Answer choice B                     |
| `option_c`        | Answer choice C                     |
| `option_d`        | Answer choice D                     |
| `correct_answer`  | Correct answer                      |
| `explanation`     | Explanation of correct answer       |
| `difficulty`      | Difficulty level                    |
| `topic`           | Topic tested                        |
| `question_number` | Order within quiz                   |
| `quizzes`         | Linked quiz                         |
| `responses`       | Linked submitted responses          |

---

## Quizzes Table

Groups generated questions into a quiz session.

| Field            | Purpose                      |
| ---------------- | ---------------------------- |
| `record_id`      | Unique quiz identifier       |
| `created_at`     | Creation timestamp           |
| `status`         | Quiz workflow stage          |
| `source`         | Quiz creation source         |
| `quiz_title`     | Quiz name                    |
| `topic`          | Quiz topic                   |
| `difficulty`     | Overall quiz difficulty      |
| `question_type`  | Included question type       |
| `question_count` | Total number of questions    |
| `questions`      | Linked questions             |
| `document`       | Linked source document       |
| `user_id`        | User taking the quiz         |
| `generation_id`  | AI generation run identifier |
| `started_at`     | Quiz start time              |
| `completed_at`   | Quiz completion time         |
| `responses`      | Linked submitted responses   |
| `performance`    | Linked performance summary   |

---

## Responses Table

Stores submitted quiz answers.

| Field                                | Purpose                      |
| ------------------------------------ | ---------------------------- |
| `record_id`                          | Unique response identifier   |
| `created_at`                         | Creation timestamp           |
| `status`                             | Response workflow stage      |
| `source`                             | Submission origin            |
| `quiz`                               | Linked quiz                  |
| `performance`                        | Linked performance record    |
| `user_id`                            | User identifier              |
| `submitted_1` through `submitted_10` | Submitted answers            |
| `correct`                            | Whether answers were correct |
| `feedback`                           | Feedback on submission       |
| `response_type`                      | Type of response             |
| `submitted_at`                       | Submission timestamp         |
| `graded_at`                          | Grading timestamp            |
| `score_awarded`                      | Points earned                |
| `missed_topic`                       | Topic answered incorrectly   |
| `created_by`                         | Record creator               |

---

## Performance Table

Stores final quiz performance summaries.

| Field              | Purpose                       |
| ------------------ | ----------------------------- |
| `record_id`        | Unique performance identifier |
| `created_at`       | Creation timestamp            |
| `status`           | Workflow stage                |
| `source`           | Record origin                 |
| `quiz`             | Linked quiz                   |
| `responses`        | Linked responses              |
| `user_id`          | User identifier               |
| `attempt_id`       | Attempt tracking              |
| `total_questions`  | Total questions               |
| `correct_count`    | Correct answers               |
| `incorrect_count`  | Incorrect answers             |
| `raw_score`        | Total score                   |
| `score_percent`    | Percentage score              |
| `weak_topics`      | Areas needing improvement     |
| `overall_feedback` | AI-generated summary feedback |
| `confidence_score` | AI confidence metric          |

---

## Status Standards

Approved workflow status values:

* `uploaded`
* `extracting`
* `cleaned`
* `chunked`
* `ready`
* `generating`
* `generated`
* `submitted`
* `graded`
* `complete`
* `error`

---

## Relationship Standards

Primary Airtable relationships:

* Documents → Questions
* Documents → Quizzes
* Questions → Quizzes
* Quizzes → Responses
* Responses → Performance
* Quizzes → Performance

---

## Workflow Rule

Every workflow should:

1. Read records based on `status`
2. Process only matching records
3. Write output to designated fields
4. Update the record `status`
5. Stop and mark `error` if processing fails

---

This version will paste into GitHub perfectly as a normal markdown file.
