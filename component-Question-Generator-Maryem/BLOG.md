# How I Built an AI That Grades Your Own Quizzes — And Everything That Broke Along the Way

The first time the grading workflow ran end to end without me touching anything, I genuinely did not believe it. I submitted a quiz answer, waited 45 seconds, and watched Airtable update itself — correct answer flagged, score written, performance record created, weak topic logged. No manual intervention. No button clicked. It just worked.

Getting to that moment took six weeks of workflows that silently failed, triggers that got stuck on the wrong records, and one automation system quietly overwriting another's work before I even knew it was happening.

---

## What We Were Building

Creating quizzes from study notes by hand takes hours. Grading open-ended responses takes even longer. Neither scales when you have multiple students, multiple subjects, and new material every week. We wanted to eliminate both. Our capstone project: a system that turns uploaded notes into a graded quiz with zero human steps in between.

Our team of three built this for our AI Integration capstone at John Jay College. I owned two components: the Question Generator (the AI core that produces the questions) and Quiz Delivery & Scoring (everything that happens after a student hits submit). My stack was n8n for workflow automation, Flowise for the LLM chain, Airtable as the database, and Groq running llama-3.3-70b-versatile as the model.

---

## The Decisions That Actually Mattered

Before writing a single workflow node, I tested three candidate models on the same four document chunks — biology, history, psychology, and intentionally bad data. Speed, output format, question quality, and failure behavior all went into the comparison.

I picked llama-3.3-70b-versatile over the faster llama-3.1-8b-instant for one specific reason: on bad input, the smaller model invented a completely unrelated Titanic question out of nowhere. For a quiz generator, accuracy matters more than raw speed. A wrong question is worse than a slow one.

I also chose Flowise as a layer between n8n and the Groq API rather than calling Groq directly from n8n. This meant I could update the prompt — change the question mix, adjust the output format, tune the temperature — without touching the automation workflow at all. That separation paid off repeatedly when the prompt needed adjusting mid-project.

---

## The Hard Part

The grading pipeline had one job: when a student submits an answer, compare it to the correct answer and write the result back to Airtable within 60 seconds.

It took three separate bugs to get there.

The first was invisible. Airtable had a native automation set to fire "when a form is submitted" — it was immediately writing to the `status` field on every new Response record. My n8n workflow polled every minute for records where `status` was blank. By the time n8n checked, Airtable's automation had already filled the field. n8n saw no blank records and skipped everything. Two systems running on the same table, racing each other, with no error thrown anywhere.

The second was a wrong table reference. The node that fetched the Question record was using the Response record's own Airtable ID as the lookup key — but pointing at the Questions table. It was searching for a Question that had the ID of a Response. It either returned nothing or returned the wrong record entirely, and then the IF node quietly routed every single answer to the "incorrect" branch. No error. Just silent wrong data.

The third was case sensitivity. A student typing `false` (lowercase) failed against `False` (capital F) stored by the model. One line fixed it — `.toLowerCase()` on both sides of the comparison — but it only showed up during live testing when a real user submitted a real answer.

None of these produced obvious error messages. All three required tracing node outputs manually, one at a time, to find where the data broke.

---

## What I Would Do Differently

I would build the error handling before building the happy path. Every bug I found was invisible until live testing because there was no logging, no error state written to Airtable, nothing that flagged when something went wrong. By the time I noticed, the trigger had already processed hundreds of records incorrectly.

I would also replace the Airtable Trigger node with a Schedule Trigger and Search Records from the start. The Airtable Trigger maintains an internal cursor — once it marks a record as seen, it will never go back for it even if the record's fields are incomplete. The Schedule plus Search approach has no cursor and queries Airtable's current state on every run. It's more reliable for any workflow where records might be created before all their fields are populated.

---

## What Actually Shipped

By the end of the project, the full pipeline ran automatically on new documents. A student submits study notes through a form, the system chunks the text, generates questions via Flowise, creates a quiz, grades every submission within 60 seconds, calculates a percentage score, flags weak topics, and writes a performance record — all without any manual step in between.

967 questions generated. 60+ responses graded across 15 users. One pipeline that runs itself.

The code and full component documentation are on GitHub: [github.com/maryem2005/ai-capstone-quiz-generator](https://github.com/maryem2005/ai-capstone-quiz-generator)

What's the hardest silent failure you've hit in an automation project? Drop it in the comments — I'm curious whether the "no error, just wrong data" pattern shows up everywhere or just in Airtable integrations.
