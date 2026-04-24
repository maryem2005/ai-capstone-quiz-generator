# Question Generator

Owner: Maryem Elgebaly

## Description

This component uses AI to generate quiz questions and answers from uploaded notes.

## Status

- Design complete
- Sample data prepared
- Initial implementation
- Testing
- Integration with other components
- Documentation complete

# Week 4: Model Comparison

Tested 4 AI models on 5 text samples to evaluate their suitability for the Question Generator component of our AI-Powered Quiz Generator capstone project.

## Models Tested

- HF Sentiment (distilbert-sst-2)
- HF Zero-Shot (bart-large-mnli)
- HF NER (bert-large-NER)
- Groq Llama 3 8B

## Finding

Groq Llama 3 performed best for the Question Generator component because it provided the strongest contextual understanding and most useful interpretation of input text.

See report.md for full analysis.




# Week 5: AutoML & No-Code Model Training

Trained a custom model and compared generic vs fine-tuned models.

## Custom Model Training
- Accuracy: 50%
- Precision: 50%
- Recall: 100%
- F1: 67%

## Fine-Tuned Model Comparison
- Generic: Sentiment model (not useful)
- Fine-Tuned A: CoLA (best)
- Fine-Tuned B: Sentiment (not useful)

## Finding
CoLA model is best for evaluating question quality because it detects grammatical correctness, while sentiment models fail to provide meaningful distinctions.


-----------------------------------------------

# AI Question Generator (AI Core) — Week 8 Update 
# AI Question Generator (AI Core)

## What It Does
The AI Question Generator is responsible for transforming document chunks into quiz questions using the Groq API (llama-3.3-70b-versatile). It reads document chunks marked as `status = 'ready'` in the Airtable `Documents` table, sends the raw text to the Groq API, and writes the generated questions to the `Questions` table. Each question includes fields like `question_text`, `question_type`, `difficulty`, `correct_answer`, and `explanation`.

## How It Connects to Other Components
- **Input:** Reads from the `Documents` table in Airtable. Specifically, it processes records with `status = 'ready'` and uses the `raw_text` and `subject` fields.
- **Output:** Writes to the `Questions` table in Airtable. Each question links back to its source document via the `document_id` field. Updates the `status` field in the `Documents` table to `loaded` or `error` after processing.

## Setup Instructions
### Prerequisites
1. **Accounts and API Keys:**
   - Groq API key: Obtain from your Groq account.
   - Airtable API key: Obtain from your Airtable account.
2. **Tools:**
   - n8n: Install and configure n8n for workflow automation.
   - Flowise: Install Flowise for building and testing LLM chains.

### Configuration
1. **Airtable Setup:**
   - Ensure the `Documents` and `Questions` tables are set up with the required fields as per the schema.
   - Create a personal Airtable API key and note the base ID for your Airtable workspace.

2. **n8n Workflow:**
   - Create an n8n workflow that:
     - Polls the `Documents` table for records with `status = 'ready'`.
     - Sends the `raw_text` and `subject` fields to the Groq API.
     - Parses the Groq API response and writes the generated questions to the `Questions` table.
     - Updates the `status` field in the `Documents` table to `loaded` or `error`.

3. **Groq API:**
   - Configure the Groq API endpoint in n8n with your API key.
   - Test the connection using a tool like Hoppscotch or Postman.

4. **Flowise (Optional):**
   - Use Flowise to build and test a custom LLM chain for confidence scoring or advanced question generation.

## How to Test It
1. **Unit Test the Groq API Connection:**
   - Use Hoppscotch or Postman to send a sample document chunk to the Groq API and verify the response format.

2. **Test the n8n Workflow:**
   - Add a test record to the `Documents` table with `status = 'ready'`.
   - Run the n8n workflow and verify:
     - The `Questions` table is populated with the correct fields.
     - The `status` field in the `Documents` table updates to `loaded` or `error`.

3. **Integration Test:**
   - Coordinate with the Ingestion component to ensure records flow from `status = 'chunked'` to `status = 'ready'`.
   - Verify that the Specialist component can read and display questions from the `Questions` table.

## Known Limitations
1. **Linked Record Formatting:** The `document_id` field must be written in Airtable's linked record format. Incorrect formatting will break the link between `Questions` and `Documents`.
2. **Options JSON Parsing:** The `options_json` field is stored as a JSON string. Downstream components must parse this correctly to display multiple-choice options.
3. **Error Handling:** The workflow does not yet handle Groq API failures gracefully. If the API is down or returns an error, the `status` field in `Documents` may not update correctly.
4. **Confidence Scoring:** Confidence-based routing and scoring are not implemented yet. This may affect the quality of generated questions.
5. **Performance:** Processing large document chunks may exceed the Groq API's response time limits.

By following these instructions, you can set up, test, and integrate the AI Question Generator into the full system. Ensure all workflows are tested thoroughly before Checkpoint 2.
