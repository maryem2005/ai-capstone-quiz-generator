# Week 7: RAG Security Knowledge Assistant — Evaluation Report

## 1. Setup Summary
- **LLM:** llama-3.3-70b-versatile via OpenAI Custom Model node (using Groq's API base URL: https://api.groq.com/openai/v1)
- **Embeddings:** sentence-transformers/all-MiniLM-L6-v2 via HuggingFace Inference Embeddings
- **Vector Store:** In-Memory Vector Store
- **Documents loaded:**
  - mitre-credential-access.txt (~1 page)
  - mitre-initial-access.txt (~2 pages)
  - mitre-lateral-movement.txt (~2 pages)

## 2. Test Results

| # | Question | Used Documents? | Quality | Notes |
|---|----------|----------------|---------|-------|
| 1 | What are common techniques for credential access? | Yes | Good | Listed keylogging, credential dumping, brute force, and password spraying with sources shown |
| 2 | How does phishing relate to initial access in the ATT&CK framework? | Yes | Good | Correctly explained phishing as an initial access vector including spearphishing variants, sources shown |
| 3 | What is lateral movement and what techniques do attackers use? | Yes | Partial | Correct definition but only listed 3 techniques — missed several others present in the document |
| 4 | What is the difference between spearphishing attachment and spearphishing link? | Yes | Good | Clearly explained the difference — attachment uses malicious files, link uses URLs to avoid attachment inspection |
| 5 | What are valid accounts and how do adversaries use them? | Yes | Good | Correctly explained valid accounts used for initial access, persistence, privilege escalation, and defense evasion |

## 3. Edge Case Observations
- **Unrelated question (weather):** The chatbot responded with "Hmm, I'm not sure" — it correctly avoided hallucinating an answer unrelated to the knowledge base.
- **Topic not in documents (latest CVEs from 2026):** The chatbot responded with "Hmm, I'm not sure" — it did not attempt to fabricate information, which shows the RAG system is properly constraining the LLM to the uploaded documents.
- **Out of scope topic (SQL injection):** The chatbot again responded with "Hmm, I'm not sure" — consistent behavior across all edge cases with no hallucination observed.

## 4. Settings Experiments
- **Temperature change:** Not tested — kept at 0.3 for consistency and factual accuracy throughout.
- **Chunk size change:** Not tested — default of 1000 produced good results so no adjustment was needed.
- **Top K change:** Not tested — default of 4 retrieved enough relevant chunks for accurate answers.

## 5. Reflection

**What surprised you about how RAG works?**
What surprised me most about RAG was the complexity of the underlying infrastructure needed to make it work. I expected a straightforward plug-and-play setup, but instead discovered that different nodes in Flowise operate on different frameworks — LangChain and LlamaIndex — which are not always compatible with each other. For example, the standard ChatGroq node was deprecated and only available under LlamaIndex, which meant it could not connect to LangChain-based chains. To work around this, I had to use the OpenAI Custom Model node with Groq's API base URL, which was an unexpected but valuable lesson in how LLM infrastructure actually works under the hood.

**How could you improve this chatbot for real-world use?**
To improve this chatbot for real-world use, I would replace the in-memory vector store with a persistent vector database such as Pinecone or Chroma, so that documents do not need to be re-processed every time the chatflow loads. I would also expand the knowledge base with more comprehensive and up-to-date documents, and fine-tune the chunk size and Top K retrieval settings to improve the accuracy and completeness of responses — particularly for questions like lateral movement techniques where the chatbot only retrieved a subset of the available information.

**How might you use RAG in your capstone project?**
For my capstone project, which focuses on an AI-powered quiz and notes question generator, RAG will play a central role in my component of the project. As the team member responsible for the AI functionality, I plan to use Flowise to build a RAG pipeline that ingests study materials — such as lecture notes and textbook excerpts — and uses them as the knowledge base for generating relevant and accurate quiz questions. This experience building the security knowledge assistant has given me a strong foundation for understanding how document ingestion, embeddings, vector retrieval, and LLM chaining all connect together, which I will apply directly to the capstone.
