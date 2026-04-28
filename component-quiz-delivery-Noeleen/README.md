
# Quiz Delivery & Scoring
**Owner:**
Noeleen Herbert 
## Description
 Display the quiz to the student and collect responses.

## Status
- [] Design complete
- [ ] Sample data prepared
- [ ] Initial implementation
- [ ] Testing
- [ ] Integration with other components
- [ ] Documentation complete

# Problem Statement: The Interactive Learning Gap
While the "Question Generator" (Maryem's component) creates high-quality study materials, the data remains static in Airtable. Students need a way to engage with this material through active recall, which is the process of being tested rather than just reading.

## The Solution
This component acts as the "front end" of the student experience. It transforms raw database rows into a dynamic conversation. It solves the problem of assessment by:

- Providing a user-friendly chat interface for quiz taking.

- Allowing for immediate feedback via explanations.

- Objectively scoring both Multiple Choice (MCQ) and Short Answer questions, ensuring students know exactly where they stand in their learning journey.

# Data IN: Integration with AI Question Generator
This component consumes data produced by Maryem’s AI Question Generator, stored in Airtable. Based on the provided dataset (Lines 13-124), the following fields are ingested via n8n to be processed by Flowise:

| Field Name | Description | Role in My Component |
| :----------: | :-----------: | :--------------------: |
| question_text | The question generated from lecture notes. | Displayed as the primary prompt to the student. |
| question_type | mcq or short_answer. | Determines if the UI shows options or a text field. |
| option_a - d | The four choices for MCQs. | Used to build the selection buttons for the student. |
| correct_answer | The verified right answer. | Used by Groq to compare against student input. |
| explanation | Contextual reasoning for the answer. | Displayed to the student after they answer to reinforce learning. |
| topic | e.g., History, Biology, Ecology. | Used to tag the final score so students see their weak areas. |


# Component Outputs: Results & Feedback Loop
My component produces three primary outputs that close the loop between the student and the database:
1. Real-Time Grading (via Groq): For MCQs, it provides instant verification.
For Short Answers, it uses Groq (Llama 3) to semantically analyze the student's text. Even if the wording isn't identical to the correct_answer, Groq determines if the concept is correct.
2. Quiz Analytics: Total Score: A final percentage calculated at the end of the session.
- Topic Mastery: A summary of performance categorized by the topic field (e.g., "You scored 100% in History, but 40% in Biology").
3. Airtable Write-Back (via n8n): The student’s specific responses and their final score are pushed back to the responses column in Airtable to track progress over time.

# Definition of "Done"
The Quiz Delivery and Scoring component is considered complete when the following four conditions are met:
- Interactive Interface: A student can initiate a session in Flowise and complete a full cycle of questions without manual intervention.
- Intelligent Scoring: The system successfully distinguishes between a correct and incorrect "Short Answer" using the Groq LLM logic.
- Automated Data Sync: Upon quiz completion, the student’s score and "Struggling Topics" automatically appear in the Airtable record.
- Feedback Delivery: For every question answered, the system correctly fetches and displays the explanation field from Maryem’s data to provide immediate student feedback.

## Technical Implementation Notes
Flowise: Used to design the logic flow (fetching records $\rightarrow$ presenting UI $\rightarrow$ capturing input).

Groq: Used as the high-speed inference engine for grading student responses.

n8n: Used as the API bridge to GET questions from Airtable and POST results back to the database.
