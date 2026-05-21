# Quiz Generator — Setup Guide

## Prerequisites
Before you begin, make sure you have accounts created for the following tools:
- [n8n Cloud](https://n8n.io) (free tier)
- [Airtable](https://airtable.com) (free tier)
- [Flowise](https://flowiseai.com) (free tier)
- [Groq](https://console.groq.com) (free tier) or Hugging Face Inference API

---

## Step 1: Set Up Airtable

1. Create a new Airtable base called **Quiz Generator**
2. Create the following tables with these fields:

**documents table**
| Field | Type |
|---|---|
| record_id | Autonumber |
| created_at | Create Time |
| status | Single Select: `unploaded`, `extracting`, `cleaned`, `chunked`, `ready`, `error`, `generated` |
| source | Single Line Text |
| raw_text | Long Text |
| clean_text | Long Text |
| chunked_sections | Long Text |
| chunk_count | Number |
| file_upload | Attachment |
| course | Single Line Text |
| topic | Single Line Text |
| questions | Link to Questions |
| created_by | Created By |


**questions table**
| Field | Type |
|---|---|
| record_id | Autonumber |
| created_at | Created Time |
| status | Single Select: `draft`, `generating`, `generated`, `approved`, `in_quiz`, `error`, `retired` |
| source | Single Line Text |
| document | Link to documents table |
| question_text | Long Text |
| question_type | Single Select: `mcq`, `true_false`, `short_answer` |
| correct_answer | Single Line Text |
| for options a-d | Single Line Text |
| difficulty | Single Select: `easy`, `medium`, `hard` |
| submitted_answer1-10 | Long Text |
| explanation | Long Text |
| topic | Single Line Text |
| quizzes | Link to Quizzes |
| question_number | Number |
| reponses | Link to responses |

*(Repeat for quizzes, responses, and performance tables — see `/docs/data-standards.md` for full field lists)*

3. Share the base with all team members as collaborators

---

## Step 2: Set Up Flowise

1. Log into Flowise and create a new chatflow called **Question Generator**
2. Add the following nodes to the canvas:
   - Document Store (for vector storage)
   - Chat model — connect your Groq API key here
   - Conversational Retrieval QA Chain
3. Save the chatflow and copy the **API endpoint URL** — you'll need this in Step 4

---

## Step 3: Set Up n8n Workflows

There are three workflows to import. In n8n, go to **Workflows → Import** and paste each one.

**Workflow 1: Document Ingestion**
- Trigger: Airtable form submission
- What it does: Watches for new records in the `documents` table where `status = ready`, chunks the text into sections, and sends them to Flowise's vector store
- After importing: Update the Airtable node with your own base ID and table name

**Workflow 2: Question Generator**
- Trigger: Scheduled 
- What it does: Finds chunked documents, calls Flowise to generate questions, writes results to the `questions` table
- After importing: Paste your Flowise API endpoint URL into the HTTP Request node

**Workflow 3: Quiz Delivery & Scoring**
- Trigger: Airtable form submission (quiz answers)
- What it does: Scores responses, writes results to `responses` and `performance` tables
- After importing: Update Airtable node credentials

---

## Step 4: Connect Your API Keys

| Tool | Where to Add It |
|---|---|
| Groq API key | Flowise → Chat model node → credentials |
| Airtable API key | n8n → Credentials → Airtable API |
| Hugging Face token (if used) | n8n → Credentials → Header Auth |

---

## Step 5: Test the Full Pipeline

Run through this in order to confirm everything is connected:

1. Open the Airtable `documents` table and paste in a short paragraph of study material, set `status = unprocessed`
2. Watch Workflow 1 fire and chunk the document — status should update to `chunked`
3. Wait for Workflow 2 to run — new rows should appear in the `questions` table
4. Submit a quiz answer via the Airtable form
5. Confirm a new row appears in `responses` with `is_correct` filled in
6. Check the `performance` table — your score should be recorded

If any step doesn't produce output, check the n8n execution log for that workflow first.

---

## Troubleshooting

**Workflow isn't firing**
- Check that your Airtable API key is saved correctly in n8n credentials
- Make sure the status field value matches exactly — `unprocessed` not `Unprocessed`

**Flowise returns an error**
- Confirm your Groq API key is active and has remaining credits
- Check that the chatflow is deployed (not just saved)

**Questions aren't writing to Airtable**
- Open the n8n execution log and look at the Airtable node output
- Verify your base ID and table name match exactly

---

## Team Contacts

| Component | Owner | 
|---|---|
| Document Ingestion | Ryan |
| Question Generator | Maryem |
| Quiz Delivery & Scoring | Noeleen |
| Integration | Maria |

---

The key things to keep in mind as you fill this in for real: be specific about field names and status values (they must match exactly what's in your Airtable base), and write the troubleshooting section based on problems you actually ran into during testing — that's what makes it genuinely useful.
