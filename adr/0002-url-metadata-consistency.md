# ADR-0002: URL Metadata Consistency and Replication Baseline

- Status: Proposed
- Date: 2026-07-29
- Owners: Niels Chiu

## Context

短網址 redirect 是全球、極端 read-heavy workload。題目要求 redirect P99 < 50 ms 與 99.99% availability，但尚未定義 create 成功後全球可讀的最大延遲、RPO/RTO，以及 destination 是否可修改。

Cosmos DB single-write + Session consistency 與 SQL single-primary + geo replica 在 replication topology 上都可能是 primary-centric asynchronous replication。真正差異在資料模型、partitioning、consistency contract、操作能力與相同 SLO 下的成本，不應只依產品名稱決策。

## Decision Drivers

- Redirect local read latency。
- Global visibility lag。
- Uniqueness 與 idempotency correctness。
- 1,825 億筆五年 metadata 的 partition/scale model。
- Region failover 的 RPO/RTO。
- Cache miss 下的 sustainable throughput。
- 相同 SLO 下的成本與操作複雜度。

## Considered Options

1. Cosmos DB single-write + Session consistency。
2. Cosmos DB Strong consistency。
3. Cosmos DB multi-region write。
4. Azure SQL single-primary + geo replicas。
5. 其他 distributed key-value store。

## Decision

目前採用 **benchmark baseline，而非最終產品綁定**：

- Mapping 預設 immutable。
- Authoritative create 使用 conditional write 保證唯一性。
- 以 multi-region key-value/NoSQL store、single-write + Session consistency 作第一個測試基準。
- 新 URL 在 local replica not found 時，可有限度 fallback authoritative region，處理 stale negative。
- 若需求要求 create acknowledgment 後全球立即可讀，重新評估 Strong consistency 或將 global publish 納入 acknowledgment。
- SQL Geo Replica 必須在相同 workload/SLO 下作對照測試。

## Consequences

### Positive

- 將低延遲 local read 與 global write visibility 分開量測。
- 利用 immutable mapping 避免 stale-positive 的 destination update 問題。
- 不把雲端元件標稱 SLA 直接當成 application SLA。

### Negative

- Session consistency 不保證不同 client/session 立即讀到新資料。
- Authoritative fallback 增加跨 region latency 與 failure path。
- 最終 database selection 尚未完成。

### Risks and Mitigations

- Stale negative：authoritative fallback、短 negative-cache TTL、create event propagation。
- Stale disabled state：短 TTL、tombstone/version、edge purge；高風險狀態 fail closed。
- Region outage data loss：forced-failover test 實測 observed RPO。
- Hot partition：以 `shortCode` 高 cardinality partition，並監控 physical partition saturation。

## Validation

分開測量：

1. End-to-end request latency。
2. `writeAckAt -> firstVisibleAtInRegion` visibility lag。
3. Maximum Sustainable Throughput。
4. Equal-cost throughput。
5. Equal-SLO cost。
6. Forced failover 的 RPO/RTO。

Cosmos consistency matrix：Strong、Bounded Staleness、Session、Consistent Prefix、Eventual。

SQL Geo Replica：primary commit latency/TPS、secondary read latency/TPS、log hardening lag、redo queue/lag、application-visible lag。

## Revisit Conditions

- 產品要求 create success 後全球立即可讀。
- Destination 必須可修改。
- Session visibility P99 超出 business SLA。
- SQL 在相同 SLO 下成本或操作性明顯更佳。
- Multi-write conflict model可被證明更簡單且可靠。
