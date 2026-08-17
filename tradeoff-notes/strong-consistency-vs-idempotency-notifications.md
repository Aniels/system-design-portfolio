# Strong Consistency vs Idempotency in Notification Delivery

## Problem

A notification system wants to avoid duplicate or unnecessary notifications. A natural question is whether the database should use strong consistency (for example, a strongly consistent distributed database) or a relational database so duplicate sends cannot occur.

The key lesson is that **database consistency and end-to-end delivery semantics are different problems**.

## Why strong consistency is not enough

A typical notification flow crosses multiple systems:

```text
DB/state transition
   -> queue/event stream
   -> worker
   -> external provider
   -> provider acknowledgement
```

Strong consistency can ensure that a particular database read/write sees the latest committed state. It does not make the entire chain atomic.

Examples of duplicate-producing failures:

1. Worker sends to provider, provider succeeds, worker crashes before persisting success.
2. Queue redelivers because acknowledgement was lost.
3. Producer commits DB state but crashes before/after publishing the queue message.
4. Provider times out even though it accepted the message.

None of these are solved solely by choosing a stronger read consistency level.

## More useful primitives

### Stable notification identity

Create a deterministic notification or delivery identity, e.g.:

```text
notificationId = hash(userId, eventId, channel, templateVersion)
```

Use it as the deduplication / idempotency boundary.

### Transactional publication boundary

When DB state change and event publication must not diverge, use a transactional outbox or equivalent durable publication mechanism.

### Idempotent worker

Before sending, determine whether the logical delivery already completed or is safely retryable.

### Provider response recording

Persist provider message ID / response where available so retries can reconcile uncertain outcomes.

### Reconciliation

For ambiguous timeout/crash windows, run reconciliation rather than assuming retry is always safe.

## Where database consistency still matters

A stronger consistency/transaction model can still be valuable for:

- preventing two concurrent writers from transitioning the same notification state incorrectly;
- enforcing uniqueness on notification identities;
- implementing transactional outbox semantics;
- ensuring a worker reads a committed delivery state before deciding whether to resend.

The database should therefore be selected based on the **transaction boundary and data model**, not just the desire to avoid duplicates.

## Cosmos DB Strong vs SQL: decision dimensions

Evaluate:

- transaction scope;
- partition-key locality;
- multi-region write/read requirements;
- latency budget;
- scale and throughput model;
- relational query/constraint needs;
- operational/cost model;
- failover expectations.

There is no general rule that “notifications require strong consistency” or “SQL guarantees no duplicates.”

## Current conclusion

For notification delivery, prioritize:

1. stable idempotency identity;
2. explicit state machine;
3. transactional publication boundary when needed;
4. provider-aware retry policy;
5. reconciliation of ambiguous outcomes.

Then choose database consistency based on the transaction/data requirements of those mechanisms.
