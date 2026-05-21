# Final Airtable Schema

## Documents Table

Stores uploaded source materials and tracks their progress through the document processing pipeline.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | ingestion | uploaded, extracting, cleaned, chunked, ready, error |
| source | text | ingestion/system | |
| file_upload | attachment | user | |
| course | text | user | |
| topic | text | user | |
| raw_text | long text | ingestion | |
| clean_text | long text | ingestion | |
| chunked_sections | long text | ingestion | |
| chunk_count | number | ingestion | |
| filename | text | system | |
| questions | linked record | AI Core | |
| created_by | text | system/user | |

---

## Questions Table

Stores all AI-generated quiz questions and their associated metadata.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | AI Core | generated, reviewed, error |
| source | text | AI Core | |
| document | linked record | AI Core | |
| question_text | long text | AI Core | |
| question_type | single select | AI Core | mcq, true_false, short_answer |
| option_a | text | AI Core | |
| option_b | text | AI Core | |
| option_c | text | AI Core | |
| option_d | text | AI Core | |
| correct_answer | text | AI Core | |
| explanation | long text | AI Core | |
| difficulty | single select | AI Core | easy, medium, hard |
| topic | text | AI Core | |
| question_number | number | system | |
| quizzes | linked record | Specialist | |
| responses | linked record | Specialist | |

---

## Quizzes Table

Groups generated questions into a quiz instance that users can complete.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | Specialist | generated, active, completed |
| source | text | system | |
| quiz_title | text | Specialist | |
| topic | text | system/AI Core | |
| difficulty | single select | system | easy, medium, hard |
| question_type | text | system | |
| question_count | number | system | |
| questions | linked record | Specialist | |
| document | linked record | system | |
| user_id | text | user/system | |
| generation_id | text | system | |
| started_at | datetime | system | |
| completed_at | datetime | system | |
| responses | linked record | Specialist | |
| performance | linked record | Specialist | |

---

## Responses Table

Stores submitted quiz answers from users and grading results.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | grading logic | submitted, graded |
| source | text | system | |
| quiz | linked record | Specialist | |
| performance | linked record | Specialist | |
| user_id | text | user/system | |
| submitted_1 | text | user | |
| submitted_2 | text | user | |
| submitted_3 | text | user | |
| submitted_4 | text | user | |
| submitted_5 | text | user | |
| submitted_6 | text | user | |
| submitted_7 | text | user | |
| submitted_8 | text | user | |
| submitted_9 | text | user | |
| submitted_10 | text | user | |
| correct | checkbox | grading logic | |
| feedback | long text | grading logic | |
| response_type | text | system | |
| submitted_at | datetime | system | |
| graded_at | datetime | grading logic | |
| score_awarded | number | grading logic | |
| missed_topic | text | grading logic | |
| created_by | text | system/user | |

---

## Performance Table

Stores summarized quiz performance metrics and AI-generated feedback.

| Field | Type | Written By | Status Values |
|------|------|------------|---------------|
| record_id | autonumber | Airtable | |
| created_at | datetime | system | |
| status | single select | grading logic | completed |
| source | text | system | |
| quiz | linked record | Specialist | |
| responses | linked record | Specialist | |
| user_id | text | system | |
| attempt_id | text | system | |
| total_questions | number | system | |
| correct_count | number | grading logic | |
| incorrect_count | number | grading logic | |
| raw_score | number | grading logic | |
| score_percent | percentage | grading logic | |
| weak_topics | long text | grading logic | |
| overall_feedback | long text | AI / grading logic | |
| confidence_score | percentage | AI logic | |
