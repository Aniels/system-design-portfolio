# Azure Hosted Agent: Session Continuity vs RAG vs Durable State

## Status

**Architecture note — product/API details require current Microsoft documentation validation.**

## Problem

The target agent needs to:

- run as a hosted Azure agent;
- accept a stable session/conversation identity so later turns can continue context;
- retrieve external knowledge through RAG;
- support tasks that may finish in one execution while still allowing future multi-turn continuation.

## Core architectural distinction

These three forms of state solve different problems:

1. **Conversation/session state** — preserves dialogue continuity.
2. **Retrieval knowledge** — retrieves facts/documents from an external index or knowledge source.
3. **Application/business state** — persists correctness-critical workflow state explicitly.

A common design mistake is treating session memory as if it were durable application state, or treating RAG as if it were conversation memory.

## Component model

```text
Client
  |
  v
Hosted Agent Runtime
  |---- Conversation / Session State
  |---- Model Deployment
  |---- Retrieval Tool ---> Search / Vector / Knowledge Index
  |---- Other Tools ------> External APIs
  |
  +---- Durable Application State Store (when correctness requires it)
```

## One-shot execution vs persistent session

A task may complete in one agent execution when all required inputs, retrieval, and tool calls fit within that execution. That does **not** eliminate the value of a session identifier:

- a later question may need conversational continuity;
- audit/replay may require a stable conversation identity;
- users may leave and return;
- the application may need to associate multiple executions with one logical conversation.

However, if a workflow has state such as approval status, payment state, job progress, or an external side effect, that state should be persisted explicitly rather than inferred only from chat history.

## RAG boundary

RAG should be treated as an external knowledge-access path:

```text
query -> retrieval -> evidence/context -> model reasoning
```

It is independent of whether the user has a session. A brand-new session may still use RAG; a long-running session may not need RAG for every turn.

## Offline feasibility

A hosted agent is not fully offline simply because retrieval content can be cached. Offline feasibility must be evaluated dependency by dependency:

- inference runtime;
- orchestration runtime;
- retrieval/index;
- identity/authentication;
- telemetry;
- external tools/APIs;
- persistent state.

If a mandatory dependency remains cloud-hosted, the end-to-end system is not fully offline.

## Design checklist

- What is the stable conversation/session identifier?
- What is the retention/expiration behavior?
- Which state is conversation-only vs business-critical?
- Does the retrieval index need per-user authorization filtering?
- What happens when retrieval is unavailable?
- Are tool calls idempotent?
- How are long-running or partially completed tool executions recovered?
- What is logged for audit without leaking sensitive content?

## Open validation items

The exact Azure AI Foundry Hosted Agent APIs, session/thread semantics, retention, tool invocation model, and service limits are version-dependent. Re-check current Microsoft documentation before turning this note into a production design.
