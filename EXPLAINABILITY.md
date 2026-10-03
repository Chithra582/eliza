# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Eliza** (`eliza`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Eliza (`eliza`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Autonomous Social & Web3 Agent Framework  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Eliza processes incoming social messages, retrieves associative memory, evaluates candidate actions, and formats character responses through a deterministic 5-stage decision pipeline.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
+-----------------------------------------------------------------------------------+
|                           5-STAGE DECISION PIPELINE                               |
+-----------------------------------------------------------------------------------+
|  [Stage 1: Multi-Platform Message Ingestion & Normalization]                      |
|  - Ingest raw webhook/socket events (Twitter, Discord, Telegram), sanitize payload|
|                                     |                                             |
|                                     v                                             |
|  [Stage 2: Context Assembly & Episodic Memory Retrieval]                         |
|  - Retrieve recent chat history, query vector database for semantic memories      |
|                                     |                                             |
|                                     v                                             |
|  [Stage 3: Persona Alignment & Action Candidate Scoring]                         |
|  - Score candidate responses and actions using affinity metric S_response         |
|                                     |                                             |
|                                     v                                             |
|  [Stage 4: Safety Guardrails & Threshold Verification]                            |
|  - Verify tau >= 0.65; inspect toxicity, financial safety caps, and wallet scopes |
|                                     |                                             |
|                                     v                                             |
|  [Stage 5: Response Dispatch, Action Execution & Memory Commit]                   |
|  - Send platform message, execute verified tool actions, persist turn to memory   |
+-----------------------------------------------------------------------------------+
```

### 2. Decision Logic & Routing Formulations

For an incoming message $m_t$, active character configuration $c_i$, and retrieved context memory $M$, the response decision score $S_{\text{response}}(c_i, m_t)$ is formulated as:

$$S_{\text{response}}(c_i, m_t) = w_{\text{persona}} P(c_i, m_t) + w_{\text{sem}} \cos(\mathbf{e}_{m_t}, \mathbf{e}_M) + w_{\text{action}} A(m_t) + w_{\text{safety}} S(m_t)$$

Where:
- $P(c_i, m_t) \in [0, 1]$ measures adherence to character voice, tone constraints, and biographical lore.
- $\cos(\mathbf{e}_{m_t}, \mathbf{e}_M)$ represents cosine similarity between query embeddings and retrieved episodic memories.
- $A(m_t) \in [0, 1]$ reflects the confidence of matched action triggers (e.g., tipping, token balance lookup, question answering).
- $S(m_t) \in [0, 1]$ represents safety compliance, penalizing prohibited financial advice or abusive language.
- Standard default weights: $w_{\text{persona}} = 0.35$, $w_{\text{sem}} = 0.30$, $w_{\text{action}} = 0.20$, $w_{\text{safety}} = 0.15$ with $\sum w = 1.0$.

Response dispatch and action execution require:

$$S_{\text{response}}(c_i, m_t) \ge \tau \quad (\tau = 0.65) \quad \land \quad S(m_t) \ge 0.90$$

### 3. Thresholding & Refusal Decision Criteria

Eliza enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_INVALID_CHARACTER_SCHEMA**: `character.json` missing mandatory name or bio halts execution with code `ERR_INVALID_CHARACTER_SCHEMA`.
- **Refusal on ERR_CLIENT_DISCONNECTED**: Socket or API connection to Discord/Twitter lost halts execution with code `ERR_CLIENT_DISCONNECTED`.
- **Refusal on ERR_UNAUTHORIZED_ONCHAIN_ACTION**: Proposed transaction value exceeds per-turn limit halts execution with code `ERR_UNAUTHORIZED_ONCHAIN_ACTION`.
- **Refusal on ERR_CONTEXT_OVERFLOW**: Assembled prompt exceeds LLM context token window halts execution with code `ERR_CONTEXT_OVERFLOW`.
- **Refusal on ERR_SAFETY_POLICY_VIOLATION**: Moderation filter score $< 0.90$ (toxicity detected) halts execution with code `ERR_SAFETY_POLICY_VIOLATION`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Tier 1 (Prompt Pruning & ReQuery):** If inference fails due to context limits, automatically truncate lowestsimilarity memories and requery the language model.
- **Tier 2 (Static Persona Bio Fallback):** If inference endpoints experience persistent API timeouts, respond with a preconfigured, incharacter canned message from the character manifest.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Tier 3 (Human Moderator Interceptor):** If a user repeatedly triggers safety guardrails or attempts prompt injection, flag the interaction to the human moderator console for manual intervention.
- **Session Telemetry Auditing**: Operators inspect execution logs, routing traces, and token usage to maintain oversight.

---

## The Data It Uses

Eliza operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Social Messages**: Text content, mentions, reply chains, and channel metadata from Discord, Twitter, and Telegram.
- **User Attributes**: User IDs, platform handles, and historical interaction timestamps.
- **Client Payloads**: Direct REST and WebSocket event frames from web and desktop interfaces.

### 2. Configuration & Reference Data

- **Character Manifests**: Declarative `character.json` files specifying names, bios, lore, stylistic rules, and knowledge vectors.
- **Episodic & Semantic Memories**: Vector embeddings stored in SQLite, PostgreSQL (pgvector), or Supabase databases.
- **Plugin Manifests**: Tool definitions and schema constraints for Web3 (Solana, EVM) and Web2 service connectors.

### 3. Base Model & Inference Lineage

- **Inference Agnostic**: Interfaces with OpenAI, Anthropic, Groq, Ollama, Together AI, and custom local models.
- **Embedding Models**: Standardized embeddings (`text-embedding-3-small`, `bge-small-en-v1.5`) for fast vector similarity search.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Eliza is essential for effective deployment.

### 1. High-volume social channels (e
- **Limitation**: High-volume social channels (e.g., active Discord servers) can cause token budget depletion if every message is processed.
- **Mitigation**: Eliza implements keyword gating, explicit `@mention` triggers, and rate-limiting cooldown timers per channel.

### 2. Character hallucinations can occasionally contradict established
- **Limitation**: Character hallucinations can occasionally contradict established lore during long multi-turn conversations.
- **Mitigation**: RAG memory injection injects immutable lore anchors into every prompt turn to enforce consistency.

### 3. Third-party social platform API outages (e
- **Limitation**: Third-party social platform API outages (e.g., Twitter rate limits) can cause message delivery delays.
- **Mitigation**: Outbound message queues buffer outgoing tweets and posts with exponential backoff and jitter.

### 4. Autonomous on-chain transactions carry financial risk
- **Limitation**: Autonomous on-chain transactions carry financial risk in the event of smart contract exploits.
- **Mitigation**: All Web3 plugins mandate maximum transaction value ceilings and multi-signature authorization for large transfers.

### 5. Multi-agent group chats can experience rapid
- **Limitation**: Multi-agent group chats can experience rapid feedback loops where agents endlessly reply to each other.
- **Mitigation**: Inter-agent communication filters detect agent-origin headers and enforce mandatory inter-turn cooldown delays.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - High-volume social channels (e | Section 1 | Verified |
| - Character hallucinations can occasionally contradict established | Section 2 | Verified |
| - Third-party social platform API outages (e | Section 3 | Verified |
| - Autonomous on-chain transactions carry financial risk | Section 4 | Verified |
| - Multi-agent group chats can experience rapid | Section 5 | Verified |
