# Kafka vs RabbitMQ — Queue vs Log Is No Longer Enough

> Validation date: 2026-09-17  
> Scope: Apache Kafka 4.3.x and RabbitMQ 4.3.x. This note focuses on architecture semantics rather than benchmark ranking.

## Problem / Context

A common shortcut is:

> RabbitMQ is a work queue; Kafka is an event log.

That mental model is still useful for understanding where the two systems came from, but it is no longer sufficient for architecture selection in 2026.

RabbitMQ now provides Streams and Super Streams: persistent, replicated append-only logs with non-destructive reads and offset-based consumption. Apache Kafka, meanwhile, made KIP-932 Share Groups production-ready in Kafka 4.2, adding cooperative consumption, per-record acknowledgement, and delivery-attempt counting for queue-like workloads.

The architecture question therefore should not be **“queue or stream?”** alone. It should be:

> What consumption, routing, ordering, replay, durability, isolation, and operational semantics does this workload require?

## Core Mental Model

### Traditional Kafka consumer groups

【Fact】Kafka topics are partitioned logs. Traditional consumer groups track progress by offsets, and partition assignment bounds parallelism inside a group. Ordering is naturally expressed within a partition.

```text
Producer
   |
   v
Kafka Topic
   +-- Partition 0: e1 -> e2 -> e3 -> ...
   +-- Partition 1: e4 -> e5 -> e6 -> ...

Consumer Group A -> its own progress
Consumer Group B -> its own progress
```

This model is particularly strong when the event history itself remains valuable after the first consumer processes it.

Typical examples:

- event-driven integration
- CDC and changelog pipelines
- analytics and telemetry
- multiple independent subscribers
- replay / reprocessing
- state reconstruction

### Traditional RabbitMQ queues

【Fact】RabbitMQ's queue-oriented model treats messages as independent deliveries. With acknowledgements enabled, consumers ACK or reject deliveries; the broker can remove acknowledged messages from a queue and redeliver unacknowledged work after failures. Exchanges and bindings provide broker-side routing.

```text
Producer
   |
   v
Exchange
   |
   +-- routing --> Queue --> Worker A
                         --> Worker B
                         --> Worker C
```

This model is particularly natural when the primary requirement is to get a piece of work completed by one of a pool of workers.

Typical examples:

- background jobs
- command dispatch
- task queues
- per-message retry / dead lettering
- broker-side routing

## Why the Old Comparison Is Incomplete

### Kafka now has queue-like consumption

【Fact】Kafka 4.2 made Share Groups production-ready. Share consumers can cooperatively consume from the same partitions, acknowledge records individually, and track delivery attempts. The number of consumers is no longer bounded by partition count in the same way as a traditional consumer group.

This means the statement:

> “Kafka cannot do queue semantics.”

is no longer correct for current Kafka.

However, Share Groups do **not** erase Kafka's log-oriented storage model. Queue-like consumption is layered on Kafka topics, so architecture trade-offs around topic retention, storage, routing, and ecosystem still remain.

### RabbitMQ now has real log semantics

【Fact】RabbitMQ Streams are persistent, replicated append-only logs with non-destructive consumer semantics. Super Streams provide a partitioned stream abstraction for higher parallelism.

This means the statement:

> “RabbitMQ messages always disappear after consumption and cannot be replayed.”

is false when RabbitMQ Streams are used. It remains broadly applicable to destructive queue consumption, not to the entire RabbitMQ platform.

## Comparison by Architecture Semantics

| Dimension | Kafka | RabbitMQ |
|---|---|---|
| Historical center of gravity | Distributed event log / streaming | Messaging / work distribution |
| Stream/log model | Native topic + partitions | Streams / Super Streams |
| Queue-like model | Share Groups (production-ready since 4.2) | Classic / Quorum queues |
| Traditional progress model | Consumer-group offsets | Per-delivery ACK/NACK |
| Replay | Natural with retained topic history | Natural with Streams; not the normal queue model |
| Ordering | Strongest within a partition | Depends on queue/stream mode and concurrency |
| Broker-side routing | Limited compared with RabbitMQ; producers normally select topics/partitions | Strong exchange / binding model |
| Independent subscribers | Separate consumer groups | Separate queues or independent stream consumers |
| Per-message work control | Improved with Share Groups | Core strength of queue model |
| Stream processing ecosystem | Strong: Kafka Streams and broad integration ecosystem | More limited compared with Kafka ecosystem |
| Long-lived event history | Strong fit; supports retention and log compaction, with tiered-storage options | Streams support retained logs, but semantics and ecosystem differ |
| Task dispatch | Now possible through Share Groups | Long-standing core use case |

The table is intentionally not a winner/loser scorecard. The correct choice depends on which semantics are first-class in the workload.

## Architecture Decision Heuristic

The most useful first question is:

> After a consumer successfully handles this data, does the data still have architectural value?

### Case A — the event remains valuable

Examples:

- `OrderCreated`
- `PaymentAuthorized`
- user activity events
- CDC records
- telemetry

Requirements may include:

- several independent downstream systems
- late-joining consumers
- replay after a bug fix
- recomputing derived state
- historical analytics

【Recommendation】Start by evaluating Kafka or RabbitMQ Streams rather than a destructive work queue.

```text
                 +--> Payment
OrderCreated ----+--> Notification
                 +--> Fraud Detection
                 +--> Analytics
                 +--> Data Lake
```

The event is not merely a job. It is part of the system's durable history.

### Case B — only completion of the work matters

Examples:

- resize an image
- render a PDF
- send one email
- execute a background job

Requirements may include:

- one worker should own each job
- explicit ACK/NACK
- isolated retry
- dead-lettering
- priorities, TTL, or flexible broker routing

【Recommendation】RabbitMQ queues remain a natural fit. Kafka Share Groups are now a valid alternative when the organization already operates Kafka or when queue-like processing should coexist with a broader Kafka event platform.

## Fan-out Is Not the Same as Competing Consumers

This distinction is critical.

### Independent fan-out

Payment, Notification, Fraud, and Analytics all need to see the same logical event.

With Kafka, separate consumer groups naturally maintain independent progress:

```text
order-events
   +--> payment-service       group
   +--> notification-service  group
   +--> fraud-service         group
   +--> analytics-service     group
```

With RabbitMQ's messaging model, an exchange can route copies into multiple queues. With RabbitMQ Streams, independent consumers can consume retained events non-destructively.

### Competing workers

A pool of workers cooperatively completes one class of jobs:

```text
image-resize jobs
      |
      +--> worker-1
      +--> worker-2
      +--> worker-3
```

RabbitMQ queues were designed around this model. Kafka Share Groups now support a similar cooperative consumption pattern.

The architecture should therefore identify whether consumers are **independent subscribers** or **competing workers** before selecting the broker topology.

## Ordering vs Parallelism

【Fact】Traditional Kafka consumer groups preserve the partition-oriented model: ordering is tied to a partition, and effective parallelism is constrained by partition assignment.

【Fact】Kafka Share Groups relax the one-consumer-per-partition constraint to support cooperative work sharing, but that comes at the expense of the strict ordered-stream processing model that traditional consumer groups are designed for.

【Inference】This reflects a general distributed-systems trade-off:

```text
stronger ordering constraints
        <->
greater scheduling freedom / parallelism
```

If business correctness depends on all events for one aggregate being processed in strict order, a keyed partitioned-log model may still be preferable to treating every record as independent work.

## Reliability: Neither Broker Gives End-to-End Exactly-Once Side Effects

A common architecture mistake is to equate broker delivery semantics with business-side-effect semantics.

Consider:

```text
message received
      |
      v
charge credit card succeeds
      |
      v
consumer crashes before broker progress is committed
```

The message may be delivered again after restart.

The same fundamental problem exists whether progress is represented by a Kafka offset, Kafka Share Group acknowledgement, or RabbitMQ acknowledgement.

The broker cannot automatically make this external sequence atomic:

```text
broker state
+ application database
+ payment provider
+ email/SMS provider
```

【Recommendation】Correctness for external side effects should normally include some combination of:

- deterministic idempotency keys
- unique constraints / atomic claims
- transactional outbox or inbox patterns
- explicit processing state
- reconciliation for ambiguous outcomes
- provider-side idempotency when available

This is particularly important for payments and notification delivery.

## Failure Modes to Evaluate

### Consumer crashes after the side effect but before acknowledgement

Risk: duplicate external action.

Mitigation: idempotency and reconciliation rather than assuming the broker can eliminate duplicates.

### Poison message

Risk: one repeatedly failing record causes retry loops or blocks progress.

Mitigation: bounded delivery attempts, dead-letter handling, operational visibility, and an explicit recovery process.

### Backlog grows faster than consumers can recover

Risk: latency grows without bound even though no data is immediately lost.

Mitigation: estimate both steady-state throughput and recovery throughput:

```text
required recovery throughput
= new incoming rate + backlog / recovery target
```

Do not size only for average throughput.

### Ordering key becomes a hotspot

Risk: one partition or logical key limits throughput.

Mitigation: challenge whether strict ordering is actually required at that granularity; otherwise repartition or redesign the aggregate boundary.

### Broker choice hides downstream bottlenecks

Risk: adding partitions or consumers does not improve end-to-end throughput because the external provider, database, or rate limit is the true bottleneck.

Mitigation: capacity planning must include the slowest downstream dependency, not only broker throughput.

## Applying the Decision to the Notification System Case

The portfolio's Notification System currently uses Kafka for campaign ingestion, fan-out, channel delivery, retry, and DLQ.

That remains defensible because the design needs:

- durable backlog
- high-throughput fan-out
- multiple asynchronous stages
- consumer-group scaling
- replay / recovery paths
- integration with explicit idempotency and reconciliation

However, the broker decision should not be justified as “Kafka supports streaming while RabbitMQ is only a queue.” That would now be technically outdated.

A better architecture justification is:

【Inference】Kafka is preferred in this case because the design is organized as a multi-stage event pipeline with durable backlog and partition-oriented scaling, and because replayable events are useful during recovery and reprocessing. RabbitMQ would remain a valid alternative, especially if broker-side routing, independent per-message scheduling, or queue-oriented operational semantics dominated the requirements.

The decision should be benchmarked against actual requirements rather than inherited technology categories.

## Current Conclusion

The reusable principle is:

> **Choose messaging infrastructure by workload semantics, not product stereotypes.**

In 2026, both Kafka and RabbitMQ can implement streaming and queue-like workloads. Their historical architectures still influence what each system makes natural, operationally simple, and ecosystem-friendly, but the old one-line rule is no longer a sufficient decision framework.

Before choosing, explicitly answer:

1. Is this data durable history or disposable work?
2. Are consumers independent subscribers or competing workers?
3. Is replay a first-class recovery mechanism?
4. What ordering scope is actually required?
5. Do we need broker-side routing, TTL, priority, or per-message control?
6. What is the retention horizon and storage model?
7. What is the steady-state and backlog-recovery throughput?
8. Where is the real bottleneck: broker, consumer, database, or external provider?
9. How are duplicate side effects prevented across system boundaries?
10. Which operational ecosystem does the team already know how to run safely?

Only after those questions are answered should `Kafka` or `RabbitMQ` appear as the architecture decision.

## References

- Apache Kafka 4.3 API — Share Consumer API: https://kafka.apache.org/43/apis/
- Apache Kafka 4.3 operations — Share Groups: https://kafka.apache.org/43/operations/basic-kafka-operations/
- Apache Kafka 4.2 release — KIP-932 production-ready Share Groups: https://kafka.apache.org/blog/2026/02/17/apache-kafka-4.2.0-release-announcement/
- RabbitMQ 4.3 Streams: https://www.rabbitmq.com/docs/stream
- RabbitMQ Reliability Guide: https://www.rabbitmq.com/docs/reliability
- RabbitMQ AMQP 0-9-1 model: https://www.rabbitmq.com/tutorials/amqp-concepts
- RabbitMQ comparison with Kafka (vendor-authored; useful as a capability map, not an independent benchmark): https://www.rabbitmq.com/docs/compare/kafka
