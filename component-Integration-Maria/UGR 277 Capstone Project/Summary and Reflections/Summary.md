# AI-Powered Quiz Generator System

## Project Overview

This project is an AI-powered quiz generation and assessment platform designed to allow users to upload study materials, automatically generate quiz questions using AI, take quizzes, submit responses, and receive grading analytics with personalized feedback.

The system integrates:

- Airtable (database + user interface)
- n8n (workflow automation)
- Flowise (AI question generation via LLM prompt engineering)

The goal was to create an end-to-end automated educational workflow capable of document ingestion, AI-powered question generation, quiz delivery, grading, and performance analysis.

---

# Architecture Image Clarification

Minor implementation refinements were made during development to improve the user submission experience; however, these changes did not materially affect the underlying system architecture, component responsibilities, or overall data flow.

Therefore, the submitted architecture diagram remains an accurate representation of the final solution.

---

# System Architecture

## Core Components

### Airtable
Used as the primary backend database and lightweight user-facing interface.

Responsible for:
- document storage
- question storage
- quiz records
- response collection
- performance analytics

---

### n8n
Used for workflow automation and orchestration.

Responsible for:
- triggering workflows
- processing uploaded records
- transforming data
- sending HTTP requests to Flowise
- grading responses
- updating Airtable records

---

### Flowise
Used for AI-powered quiz generation.

Responsible for:
- consuming processed study material
- generating quiz questions
- creating answer options
- generating explanations
- assigning difficulty metadata

---

