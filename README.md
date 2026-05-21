# AI-Powered Quiz Generator

An end-to-end educational AI system that transforms uploaded study materials into AI-generated quizzes, automatically evaluates responses, and delivers personalized performance feedback.

---

## Demo Video

<p align="center">
  <a href="https://youtu.be/MMyusKu4cQ8?si=wFM0bkBQcZDNGmra">
    <img src="https://img.youtube.com/vi/MMyusKu4cQ8/maxresdefault.jpg" alt="Watch Project Demo" width="850">
  </a>
</p>

<p align="center">
  <strong>Click above to watch the full project walkthrough</strong>
</p>

---

## System Architecture

## System Architecture

<p align="center">
  <img src="./Showcase/architecture-diagram.png" alt="AI Quiz Generator Architecture" width="1000">
</p>

---

## Component Documentation

Explore each major project component:

- 📥 [Ryan — Data Ingestion](./componet-Data-Ingestion-Ryan)
- 🧠 [Maryem — AI Question Generation](./component-Question-Generator-Maryem)
- 🔗 [Maria — Integration / Workflow Orchestration](./component-Integration-Maria)

---

## Dashboard Screenshots

<p align="center">
  <img src="./Showcase/dashboard-documents.png" alt="Documents Dashboard" width="300">
  <img src="./Showcase/dashboard-questions-quizzes.png" alt="Questions and Quizzes Dashboard" width="300">
  <img src="./Showcase/dashboard-performance.png" alt="Performance Dashboard" width="300">
</p>
---

## Team Contributions

| Team Member | GitHub | Role |
|-----------|--------|------|
| Maria Shirin | MarialsCoding | Integration / Workflow Orchestration |
| Maryem Elgebaly | maryem2005 | AI Question Generation |
| Ryan Maca | RyanMaca01 | Data Ingestion |

---

## Who Built What

### Maria — Integration / Workflow Orchestration
- Connected all independent workflows into one complete end-to-end system
- Designed workflow orchestration logic between ingestion, AI generation, and quiz delivery
- Validated Airtable schema relationships and linked record architecture
- Debugged trigger execution, workflow handoffs, and integration issues
- Conducted end-to-end testing and final system validation

### Maryem — AI Core / Question Generation
- Built AI-powered quiz generation workflows
- Integrated Flowise with Groq API for LLM-powered content generation
- Engineered prompts for quiz creation
- Implemented multiple choice, true/false, and short-answer generation logic
- Structured AI-generated outputs for Airtable integration

### Ryan — Data Ingestion
- Built the document ingestion pipeline
- Implemented Airtable upload intake workflows
- Developed text extraction and preprocessing logic
- Structured cleaned document content for downstream AI processing
- Prepared chunked content for question generation workflows

---
## Team Member Profiles

<table align="center">
  <tr>
    <td align="center" width="250">
      <a href="https://github.com/MariaIsCoding">
        <strong>Maria Shirin</strong>
      </a>
      <br>
      Integration / Workflow Orchestration
    </td>
    <td align="center" width="250">
      <a href="https://github.com/maryem2005">
        <strong>Maryem Elgebaly</strong>
      </a>
      <br>
      AI Question Generation
    </td>
    <td align="center" width="250">
      <a href="https://github.com/RyanMaca01">
        <strong>Ryan Maca</strong>
      </a>
      <br>
      Data Ingestion
    </td>
  </tr>
</table>

## Project Summary

This capstone project demonstrates a modular educational AI workflow that enables users to upload study materials, automatically generate AI-powered quizzes, complete assessments, receive automated grading, and view performance analytics through an integrated workflow architecture.
