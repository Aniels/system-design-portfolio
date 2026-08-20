# Agent Architecture: Session State vs RAG vs Durable Business State

## Status

Architecture note. The core principle is product-agnostic; Azure Hosted Agent is one possible implementation context and product/API details must be revalidated against current documentation before production use.

## Problem

Agent systems often mix three different kinds of state:

1. **Conversation/session state** — preserves dialogue continuity.
2. **Retrieval knowledge** — fetches external facts/documents through RAG.
3. **Durable business state** — persists correctness-critical workflow state explicitly.

Treating these as interchangeable creates hidden correctness and recovery risks.

## Core architectural distinction

```text
Client
  |
  v
Agent Runtime
  |---- Conversation / Session State
  |---- Model
  |---- Retrieval Tool ---> Search / Vector / Knowledge Index
  |---- Other Tools ------> External APIs
  |
  +---- Durable Application State Store
```

### Session state

Use session/conversation state for conversational continuity, follow-up turns, context reconstruction, and user experience.

Do not assume it is the authoritative store for workflow correctness.

### RAG

RAG is a knowledge-access path:

```text
query -> retrieval -> evidence/context -> model reasoning
```

A new session may still use RAG, and a long-running session may not need retrieval on every turn.

### Durable business state

Persist explicit state when correctness depends on it, for example:

- approval status;
- payment state;
- job progress;
- workflow checkpoints;
- external side effects;
- idempotency records.

This state should be independently queryable, recoverable, and auditable rather than inferred only from chat history.

## One-shot execution vs persistent session

A task may complete in one execution if all required inputs, retrieval, and tool calls fit within that run. That does not make persistent session identity useless: later questions, audit/replay, and user return flows may still need a stable conversation identity.

The important boundary is not “single turn vs multi-turn,” but **which state must survive independently of the agent runtime for correctness**.

## Failure modes

- Session expiration removes context that the application incorrectly treated as durable state.
- Retrieval is unavailable, causing a model to reason without required evidence.
- Tool retries duplicate external side effects when idempotency is missing.
- Business state diverges from conversation state after partial failure.
- Authorization filters are applied to chat context but not to retrieval results.
- Audit logs capture sensitive conversational content without an explicit retention policy.

## Design checklist

- What is the stable conversation/session identifier?
- What is its retention and expiration behavior?
- Which state is conversational only?
- Which state is correctness-critical and must be persisted explicitly?
- Does retrieval enforce per-user or per-tenant authorization?
- What happens when retrieval is unavailable?
- Are tool calls idempotent?
- How are partially completed tool executions recovered?
- What must be logged for audit, and what must not be retained?

## Current conclusion

**Session memory, RAG, and durable business state solve different problems and should be modeled as separate architectural concerns.** Session state improves continuity, RAG supplies external knowledge, and durable application state protects workflow correctness and recovery.

Azure AI Foundry / Hosted Agent can be used as an implementation example, but exact session/thread APIs, retention, tool semantics, and service limits are product-version-dependent and should not be generalized into the architecture principle itself.
