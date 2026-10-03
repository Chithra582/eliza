---
name: "autonomous-memory-and-rag"
description: "Manage semantic vector embeddings and episodic memory retrieval across interactions."
---

# Autonomous Memory and RAG Skill

Provides continuous conversational memory and knowledge retrieval.

## Core Capabilities
- Encodes message turns into dense vector embeddings.
- Retrieves high-relevance context from vector stores (SQLite, Postgres, Supabase).
- Prunes and compacts older context to satisfy LLM token constraints.
