# CIP Bot — LLM-Powered Protocol Assistant

**Built at Cisco's internal HackAIthon · Sep–Nov 2025**

> **A note on this repo:** CIP Bot was built against internal Cisco systems, proprietary test infrastructure, and real router/switch hardware, so the original implementation isn't something I can open-source. This README is a technical write-up of the architecture, design decisions, and results — written so the engineering is fully inspectable even without the underlying code.

---

## The Problem

Engineers debugging CIP/ODVA industrial protocol issues were losing significant time doing two manual, repetitive things:

1. **Cross-referencing spec documents** — manually searching through dense protocol specifications to answer questions like "what does status code 0x0E mean in this context?"
2. **Reading raw test logs line-by-line** — manually scanning tens of thousands of lines of test output to find the handful of lines that actually mattered for a given failure.

Both tasks were slow, error-prone under time pressure, and didn't scale as the spec surface and log volume grew.

## What I Built

A two-part internal tool, integrated directly into the team's existing chat platform (no new tool to learn, no context-switching):

### 1. A Retrieval-Augmented Generation (RAG) pipeline for spec Q&A

- Indexed **thousands of spec chunks** from CIP/ODVA protocol documentation into a vector database.
- Engineers ask natural-language questions in chat ("What's the required response for a Forward_Open failure with status X?") and get spec-grounded answers — not hallucinated summaries, but answers traceable back to the exact source chunk.
- The core engineering problem here wasn't the LLM — it was **retrieval quality**: getting the right chunk out of thousands of indexed documents, fast and accurately, mattered far more than prompt engineering. Chunking strategy (size, overlap, and preserving section context) had the single biggest impact on answer quality.

### 2. A high-speed log-decoding engine

- Parses **very large test logs (tens of thousands of lines) in under a second**, turning dense raw traces into something a human can actually read and query.
- Designed to be the first thing an engineer reaches for instead of manually grep-ing or scrolling through raw output.

### 3. Live integration with test infrastructure

- Connected the bot to real-time routers and switches so it could support **live feature-testing**, not just static document lookup — engineers could ask about current test state, not just historical specs.

## Architecture (high level)

```
Engineer question (team chat)
        │
        ▼
  Query embedding
        │
        ▼
  Vector search ──► Top-k relevant spec chunks
        │
        ▼
  LLM generates grounded answer
        │
        ▼
  Response posted back in chat thread

Raw test log ──► Log-decoding engine ──► Structured, queryable output
```

## Results

| Metric | Result |
|---|---|
| Spec chunks indexed | Thousands |
| Log parsing speed | Tens of thousands of lines in under a second |
| Manual spec cross-referencing | Eliminated |
| Workflow integration | Native (team chat tool, no new tool to learn) |

## Tech Stack

- **Retrieval-Augmented Generation (RAG)**
- **Vector database** — storing and searching spec chunk embeddings
- **LLM (GPT-class model)** — answer generation, grounded in retrieved context
- **Chat platform API** — integration into the team's existing workflow
- **Python** — pipeline and log-decoding engine

## Key Takeaway

The hardest part of applying LLMs to internal engineering tools isn't the model — it's the retrieval and grounding. A well-tuned chunking and retrieval strategy will outperform a more "powerful" model with weak retrieval, every time. The log-decoding engine's speed also mattered as much as the RAG pipeline's accuracy — engineers trust a tool more when it's fast enough to be part of their actual debugging flow, not a side errand.

---

*Built solo at Cisco's internal HackAIthon. Original code and data are internal to Cisco and not included in this repository.*
