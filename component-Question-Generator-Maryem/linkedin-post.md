Three automation systems. One table. Zero errors thrown — just wrong data, everywhere, silently. 👀

I just wrapped my AI Integration capstone at John Jay College and honestly?? The AI was the easy part.

I built two components of an AI-Powered Quiz Generator — the Question Generator (upload study notes → Flowise LLM chain on Groq → quiz questions auto-generated in Airtable ✨) and the Quiz Delivery & Scoring pipeline (student submits an answer → graded within 60 seconds → performance tracked automatically, zero humans involved 🤖).

Sounds clean. It was NOT clean.

The hardest part was getting n8n, Airtable's native automations, and Flowise to play nice together without racing each other or silently dropping data. My favorite bug: Airtable's own automation was filling a status field milliseconds before my n8n workflow could check it — so n8n polled, saw nothing to grade, and skipped every single new submission. No error message. No alert. Just complete silence while I stared at the screen wondering why nothing was happening 💀

I learned to trace every bug through node outputs one at a time until I found exactly where reality stopped matching expectation. Genuinely one of the most useful debugging skills I've picked up.

Final numbers: 967 questions generated. 60+ responses graded. 15 users. One pipeline that runs itself.

A huge thank you to the IIE program 
created by the John Jay Career Learning Lab for making this possible. 
This program gave me the tools, structure, and real-world project 
experience to actually build something I'm proud of. 🙏


Full write-up + all the docs here 👇
https://github.com/maryem2005/ai-capstone-quiz-generator/blob/main/component-Question-Generator-Maryem/BLOG.md

What's the most frustrating silent failure you've hit in an automation project? Drop it in the comments — I'll go first 👇

#AIIntegration #n8n #Flowise #Airtable #NoCode #JohnJayCollege
