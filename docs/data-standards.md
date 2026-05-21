# Data Standards

## Naming Conventions

All Airtable fields use:

- `snake_case`
- lowercase status values
- `_at` for date/time fields
- clear linked-record field names
- no spaces or capital letters in field names

Example:

```text
created_at
quiz_title
score_percent
confidence_score
````

## Universal Fields

Each main Airtable table includes these shared fields:

| Field        | Purpose                                    |
| ------------ | ------------------------------------------ |
| `record_id`  | Unique auto-generated identifier           |
| `created_at` | Timestamp for when the record was created  |
| `status`     | Tracks the record’s current workflow stage |
| `source`     | Identifies where the record came from      |

## Core Tables

This project uses five main Airtable tables:

```text
Documents
Questions
Quizzes
Responses
Performance
```

## Documents Table

Stores uploaded study materials and tracks document processing.

| Field              | Purpose                           |
| ------------------ | --------------------------------- |
| `record_id`        | Unique document ID                |
| `created_at`       | Date/time created                 |
| `status`           | Processing stage                  |
| `source`           | Origin of the document            |
| `file_upload`      | Uploaded file                     |
| `course`           | Course name                       |
| `topic`            | Document topic                    |
| `raw_text`         | Original extracted text           |
| `clean_text`       | Cleaned text                      |
| `chunked_sections` | Text split into AI-ready sections |
| `chunk_count`      | Number of chunks                  |
| `filename`         | Original file name                |
| `questions`        | Linked generated questions        |
| `created_by`       | Person who uploaded it            |

## Questions Table

Stores AI-generated quiz questions.

| Field             | Purpose                             |
| ----------------- | ----------------------------------- |
| `record_id`       | Unique question ID                  |
| `created_at`      | Date/time created                   |
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
| `explanation`     | Explanation for the correct answer  |
| `difficulty`      | Difficulty level                    |
| `topic`           | Topic tested                        |
| `question_number` | Question order                      |
| `quizzes`         | Linked quiz                         |
| `responses`       | Linked submitted responses          |

## Quizzes Table

Groups questions into a quiz.

| Field            | Purpose                    |
| ---------------- | -------------------------- |
| `record_id`      | Unique quiz ID             |
| `created_at`     | Date/time created          |
| `status`         | Quiz stage                 |
| `source`         | How quiz was created       |
| `quiz_title`     | Quiz name                  |
| `topic`          | Quiz topic                 |
| `difficulty`     | Overall difficulty         |
| `question_type`  | Type of questions used     |
| `question_count` | Number of questions        |
| `questions`      | Linked questions           |
| `document`       | Linked source document     |
| `user_id`        | User taking the quiz       |
| `generation_id`  | AI generation run ID       |
| `started_at`     | Quiz start time            |
| `completed_at`   | Quiz completion time       |
| `responses`      | Linked responses           |
| `performance`    | Linked performance summary |

## Responses Table

Stores the user’s submitted quiz answers.

| Field                        | Purpose                          |
| ---------------------------- | -------------------------------- |
| `record_id`                  | Unique response ID               |
| `created_at`                 | Date/time created                |
| `status`                     | Response stage                   |
| `source`                     | Submission source                |
| `quiz`                       | Linked quiz                      |
| `performance`                | Linked performance record        |
| `user_id`                    | User who submitted answers       |
| `submitted_1`–`submitted_10` | User answers for each question   |
| `correct`                    | Whether the response was correct |
| `feedback`                   | Feedback on answers              |
| `response_type`              | Type of response                 |
| `submitted_at`               | Time submitted                   |
| `graded_at`                  | Time graded                      |
| `score_awarded`              | Points awarded                   |
| `missed_topic`               | Topic missed                     |
| `created_by`                 | Person who created the record    |

## Performance Table

Stores final quiz results and feedback.

| Field              | Purpose                             |
| ------------------ | ----------------------------------- |
| `record_id`        | Unique performance ID               |
| `created_at`       | Date/time created                   |
| `status`           | Performance record stage            |
| `source`           | Where performance data came from    |
| `quiz`             | Linked quiz                         |
| `responses`        | Linked responses                    |
| `user_id`          | User receiving results              |
| `attempt_id`       | Tracks multiple attempts            |
| `total_questions`  | Total questions                     |
| `correct_count`    | Number correct                      |
| `incorrect_count`  | Number incorrect                    |
| `raw_score`        | Total points earned                 |
| `score_percent`    | Final percentage score              |
| `weak_topics`      | Topics needing improvement          |
| `overall_feedback` | AI-generated performance feedback   |
| `confidence_score` | AI confidence in the grading output |

## Status Standards

Status values should stay lowercase and consistent across workflows.

Recommended values:

text
uploaded
extracting
cleaned
chunked
ready
generating
generated
submitted
graded
complete
error


## Relationship Standards

The main record relationships are:

text
Documents → Questions
Documents → Quizzes
Questions → Quizzes
Quizzes → Responses
Responses → Performance
Quizzes → Performance

## Workflow Rule

Each workflow should:

1. Read records with the correct `status`
2. Process only those records
3. Write results to the correct fields
4. Update the `status`
5. Stop or mark `error` if processing fails



