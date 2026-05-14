# Week 7: RAG Security Knowledge Assistant — Evaluation Report

## 1. Setup Summary

- **LLM:** `llama-3.3-70b-versatile` via Groq
-**Embeddings:** `sentence-transformers/distilbert-base-nli-mean-tokens` via HuggingFace Inference
- **Vector Store:** In-Memory Vector Store
- **Documents Loaded:**
  - `mitre-initial-access.txt` (~technique descriptions + procedure examples)
  - `mitre-credential-access.txt` (~technique descriptions + procedure examples)
  - `mitre-lateral-movement.txt` (~technique descriptions + procedure examples)

These documents were created using MITRE ATT&CK Enterprise tactic pages by selecting relevant techniques and copying the description and procedure example content into text files.

---

## 2. Test Results

| # | Question | Used Documents? | Quality | Notes |
|---|----------|----------------|---------|-------|
| 1 | What are common techniques for credential access according to MITRE? | Yes | Partial | The chatbot referenced uploaded credential access content, but the answer was somewhat narrow and focused on only a few examples rather than giving a broader overview of common credential access techniques from the documents. |
| 2 | How does phishing relate to initial access in the ATT&CK framework? | Yes | Good | The chatbot directly referenced the uploaded MITRE initial access content and accurately explained phishing as a common initial access technique, including spearphishing and social engineering concepts from the documents. |
| 3 | What is lateral movement and what techniques do attackers use? | Yes | Partial | The chatbot correctly defined lateral movement and referenced relevant concepts from the uploaded documents, but the response lacked depth and did not fully cover multiple attacker techniques present in the source material. |
| 4 | How can adversaries abuse valid accounts for initial access? | Yes | Partial | The chatbot gave a generally relevant answer about abusing valid accounts, but part of the response drifted into broader attacker behavior beyond the exact scope of the uploaded documents, making the answer only partially grounded. |
| 5 | What is the difference between brute force and credentials from password stores? | Yes | Good | The chatbot accurately distinguished between brute force attacks and credential theft from password stores using concepts directly from the uploaded credential access documents, making the response both relevant and accurate. |

---

## 3. Edge Case Observations

### Unrelated Question
When asked an unrelated question outside the uploaded cybersecurity knowledge base, the chatbot responded with uncertainty (essentially indicating that it was not sure how to answer). This suggests the retrieval system was appropriately limited by the uploaded documents rather than confidently generating unsupported information. It also demonstrated that the RAG pipeline performs best when the query aligns with the provided source material.

### Topic Not in Documents
When asked about phishing prevention strategies, the chatbot correctly recognized that the uploaded documents focused on attack techniques rather than defensive mitigation. However, it still attempted to provide general cybersecurity recommendations, showing mild hallucination behavior instead of strictly limiting itself to document-based knowledge.

--

## 5. Reflection

### What surprised you about how RAG works?
One surprising aspect was how much the chatbot's quality depended on the uploaded document scope. When the answer existed clearly in the documents, responses were highly accurate and grounded. However, when the information was only partially available, the chatbot sometimes filled in missing details using general model knowledge instead of strictly relying on retrieval.

### How could you improve this chatbot for real-world use?
For real-world deployment, I would improve the chatbot by:
- using a persistent vector database instead of in-memory storage
- expanding the knowledge base with more comprehensive MITRE ATT&CK documentation
- adding defensive mitigation documentation (such as MITRE mitigations or NIST controls)
- improving prompt engineering to reduce hallucinations when information is missing
- implementing access controls and logging for security monitoring

### How might you use RAG in your capstone project?
RAG could be highly useful in my capstone project for building an AI-powered cybersecurity assistant capable of retrieving trusted security knowledge from structured documentation. For example, it could support threat analysis by querying ATT&CK techniques, mapping adversary behavior, or assisting analysts in identifying likely attack patterns based on uploaded security intelligence sources.

