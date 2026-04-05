
[week-07-report.md](https://github.com/user-attachments/files/26486289/week-07-report.md)
# Week 7: RAG Security Knowledge Assistant — Evaluation Report

## 1. Setup Summary
- **LLM:** mistral-small-latest via Mistral AI
- **Embeddings:** sentence-transformers/all-MiniLM-L6-v2 via HuggingFace Inference API
- **Vector Store:** In-Memory Vector Store
- **Documents loaded:**
  - mitre-initial-access.txt (~2 pages) — MITRE ATT&CK Initial Access techniques (TA0001)
  - mitre-credential-access.txt (~2 pages) — MITRE ATT&CK Credential Access techniques
  - mitre-lateral-movement.txt (~2 pages) — MITRE ATT&CK Lateral Movement techniques

## 2. Test Results

| # | Question | Used Documents? | Quality | Notes |
|---|----------|----------------|---------|-------|
| 1 | What are common techniques for credential access according to MITRE? | Yes | Good | Correctly identified techniques like Input Capture, LSASS Memory, SAM database, and DCSync from uploaded documents |
| 2 | How does phishing relate to initial access in the ATT&CK framework? | Yes | Good | Detailed response covering spearphishing attachment, link, and via service — all pulled from uploaded documents |
| 3 | What is lateral movement and what techniques do attackers use? | Yes | Good | Accurately described lateral movement tactics and techniques from the uploaded MITRE document |
| 4 | What does the NIST framework recommend for the Detect function? | No | Good | Correctly responded with "I'm not sure" since no NIST documents were uploaded — no hallucination occurred |
| 5 | What is the difference between spearphishing attachment and spearphishing link? | Yes | Good | Provided a detailed comparison including delivery method, user action required, payload type, and detection difficulty |

## 3. Edge Case Observations
- **Unrelated question:** Asked "What is the weather like today?" — the chatbot responded with "I'm not sure" and did not attempt to make up an answer. This shows the RAG system correctly limits responses to the uploaded knowledge base.
- **Topic not in documents:** Asked "What are the latest CVEs from 2026?" — the chatbot responded with "I'm not sure" rather than hallucinating CVE data. This is the correct behavior for a RAG system.
- **Specific topic not uploaded:** Asked "What does the NIST framework recommend for incident response?" — again responded with "I'm not sure" since no NIST documents were uploaded. No hallucination occurred.

## 4. Settings Experiments
- **Temperature change:** Not tested — kept at 0.3 throughout to maintain factual and consistent responses appropriate for a security knowledge assistant.
- **Chunk size change:** Not tested — kept at default 1000 characters with 200 character overlap which produced good results.
- **Top K change:** Not tested — kept at default value of 4 which provided sufficient context for answering questions.

## 5. Reflection
- **What surprised you about how RAG works?** I was surprised by how well the chatbot stayed within the boundaries of the uploaded documents. When asked about topics not in the knowledge base (like NIST or current CVEs), it simply said it didn't know rather than making something up. I expected it to hallucinate more often, but the RAG retrieval system did a good job of grounding the responses in the actual uploaded content.

- **How could you improve this chatbot for real-world use?** To improve this chatbot for real-world use I would switch from an in-memory vector store to a persistent one like Pinecone or Chroma so the documents don't need to be reloaded every session. I would also upload more comprehensive documents covering all MITRE ATT&CK tactics and include NIST framework documents to broaden the knowledge base. Adding a better prompt template to guide the model's response format would also help.

- **How might you use RAG in your capstone project?** I could use RAG in my capstone project to build a knowledge assistant that answers questions based on domain-specific documents relevant to my field. Instead of relying on the LLM's general training data, RAG would allow the chatbot to reference specific procedures, guidelines, or frameworks uploaded as documents — making the answers more accurate and relevant to the specific use case.
