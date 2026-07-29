# Capacity Estimation Cheatsheet

## Time conversions

- 1 day = 86,400 seconds
- Average QPS = daily requests / 86,400
- Peak QPS should be stated as an assumption, commonly 2–10× average depending on workload

## Storage

```text
Daily storage = writes/day × average record size × replication factor
Annual storage = daily storage × 365
```

Include indexes, metadata, tombstones, logs, backups and growth margin. Do not estimate only the raw payload.

## Bandwidth

```text
Ingress bandwidth = write QPS × request size
Egress bandwidth = read QPS × response size
```

Convert bytes/second to bits/second by multiplying by 8.

## Cache sizing

```text
Working set = active keys × average cached object size
Required memory ≈ working set / target utilization
```

Reserve memory for replication, fragmentation, eviction metadata and failover.

## Queue backlog

```text
Backlog growth/second = producer rate - consumer rate
Drain time = backlog / spare consumer throughput
```

Always define the maximum tolerable lag and autoscaling delay.

## Availability

| Availability | Approximate downtime/year |
|---:|---:|
| 99% | 3.65 days |
| 99.9% | 8.76 hours |
| 99.99% | 52.6 minutes |
| 99.999% | 5.26 minutes |

End-to-end availability for serial dependencies is approximately the product of component availability. Redundancy does not automatically improve availability unless failure domains are independent.

## Latency budget

Break the SLO into explicit budgets:

```text
Edge + authentication + application + cache/database + network + safety margin
```

Use percentile targets such as p50, p95, p99 and p99.9 rather than averages alone.

## Estimation discipline

1. State every assumption.
2. Preserve units throughout calculations.
3. Calculate average and peak separately.
4. Identify the first bottleneck.
5. Add headroom and explain why.
6. Validate assumptions with load tests before claiming production capacity.
