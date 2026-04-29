# Advanced RAG System Design on GCP

## Overview
This document describes a production-grade Retrieval-Augmented Generation (RAG) system on GCP using:
- GKE (serving layer)
- Vertex AI (LLM + embeddings)
- BigQuery / Matching Engine (vector search)
- GCS (document storage)

---

## Architecture Diagram

![RAG Architecture](a_clean_technical_architecture_diagram_system_de.png)

---

## 1. Parent-Child Chunking

### Concept
- Parent: Large document (Jira ticket / Wiki page)
- Child: Smaller chunks (200–400 tokens)

### Flow
- Split document → Parent chunks (~2000 tokens)
- Split parent → Child chunks (~300 tokens)
- Store mapping: child_id → parent_id

### Benefit
- High retrieval precision (child)
- Rich context (parent)

---

## 2. Embedding

- Use Vertex AI embedding model
- Generate embeddings ONLY for child chunks

### Storage
- Vector DB: child embeddings
- Metadata: parent_id, tags, status

---

## 3. Hybrid Search

### Combine:
- Dense search (vector similarity)
- Sparse search (BM25 / keyword)

### Flow
Query → Embedding → Vector search  
Query → Keyword → BM25  
→ Merge results (RRF)

### Benefit
- Better recall + precision

---

## 4. Context Builder

### Problem
Too much context → token overflow

### Solution
- Fetch parent docs
- Apply:
  - Deduplication
  - Compression
  - Sliding window

---

## 5. Guardrails

### Flow
Generate → Verify → Decide

### Decision
- PASS → return
- RETRY → re-query
- FAIL → safe fallback

---

## 6. Hallucination Handling

### Detection
- LLM-as-judge
- Faithfulness score
- Context grounding

### Prevention
- Strict prompting
- Better retrieval
- Citation enforcement

---

## 7. End-to-End Flow

User Query  
→ Query Rewrite  
→ Hybrid Retrieval  
→ Child chunks  
→ Parent fetch  
→ Context builder  
→ LLM  
→ Guardrails  
→ Final Answer  

---

## 8. Key Takeaways

- Retrieval quality > LLM choice
- Parent-child improves accuracy significantly
- Guardrails are mandatory for production

