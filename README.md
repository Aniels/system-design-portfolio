# System Design Portfolio

這個 repository 用來累積可公開展示的系統設計案例，作為應徵 **Solution Architect、Cloud Architect、System Architect、Staff/Principal Engineer** 等職位時的作品集。

作品集不只呈現架構圖，也記錄需求假設、容量估算、資料模型、API、可靠性、資安、成本與技術取捨，讓每個設計都能被檢驗與討論。

## Portfolio Roadmap

| # | Case study | 核心能力 | 狀態 | 完成度 |
|---|---|---|---|---:|
| 01 | [URL Shortener](case-studies/01-url-shortener/README.md) | Read-heavy、ID generation、multi-level cache、consistency benchmark | Design complete; validation pending | 75% |
| 02 | [Global Rate Limiter](case-studies/02-global-rate-limiter/README.md) | Distributed quota、lease、failure policy | In progress | TBD |
| 03 | [Notification System](case-studies/03-notification-system/README.md) | Event-driven、retry、idempotency | In progress | TBD |
| 04 | [Chat System](case-studies/04-chat-system/README.md) | WebSocket、ordering、presence | Planned | 0% |
| 05 | [Cloud Storage](case-studies/05-cloud-storage/README.md) | Chunking、sync、metadata | Planned | 0% |
| 06 | [Social Feed](case-studies/06-social-feed/README.md) | Fan-out、timeline、celebrity problem | Planned | 0% |
| 07 | [Video Streaming](case-studies/07-video-streaming/README.md) | CDN、transcoding、adaptive bitrate | Planned | 0% |
| 08 | [Ride-hailing](case-studies/08-ride-hailing/README.md) | Geospatial index、matching、realtime | Planned | 0% |
| 09 | [Distributed Job Scheduler](case-studies/09-distributed-job-scheduler/README.md) | Lease、leader election、execution semantics | Planned | 0% |
| 10 | [Global Load Balancer](case-studies/10-global-load-balancer/README.md) | Anycast、DNS、health probe、failover | In progress | TBD |

> URL Shortener 的 75% 是 repository 內部進度標記：設計文件已完成，但 consistency/database benchmark、cache trace simulation、RPO/RTO 與 cost model 尚未驗證，不代表 production readiness。

## Repository Structure

```text
.
├── case-studies/            # 每題完整設計與面試版摘要
├── architecture-diagrams/   # Mermaid、draw.io、SVG 與 PNG
├── tradeoff-notes/          # 跨案例的技術選型與取捨
├── adr/                     # Architecture Decision Records
├── docs/                    # 模板、估算速查與檢核表
└── .github/                 # Issue template 與協作設定
```

## Definition of Done

每個 case study 至少應包含：

1. 問題定義、功能與非功能需求
2. 明確的規模假設與容量估算
3. API、資料模型及 partition key
4. High-level architecture 與關鍵資料流
5. 一致性、可用性、延遲與成本取捨
6. Failure modes、重試、冪等與災難復原
7. Security、privacy、observability 與測試方法
8. Bottleneck、替代方案與演進路線
9. 五分鐘面試講解版

## Recommended Workflow

1. 從 [System Design Template](docs/design-template.md) 複製章節。
2. 先寫需求與 SLO，再選技術。
3. 使用 [Capacity Estimation Cheatsheet](docs/capacity-estimation-cheatsheet.md) 驗證量級。
4. 將重要選擇記錄成 ADR。
5. 用 [Interview Checklist](docs/interview-checklist.md) 進行自評。
6. 透過 GitHub Issue 追蹤缺口，每個 case study 使用獨立 PR。

## Current Priorities

目前優先把已討論過的四題整理成完整案例：

1. URL Shortener：完成 benchmark、RPO/RTO 與 cost validation。
2. Global Rate Limiter。
3. Notification System。
4. Global Load Balancer。

## Reusable Notes

- [Multi-level Cache and Consistency](tradeoff-notes/multi-level-cache-and-consistency.md)
- [APIM AI Gateway × Foundry Priority Processing](tradeoff-notes/apim-ai-gateway-priority-processing.md)
- [Azure Hosted Agent — Session + RAG](tradeoff-notes/azure-hosted-agent-session-rag.md)
- [Strong Consistency vs Idempotency in Notification Delivery](tradeoff-notes/strong-consistency-vs-idempotency-notifications.md)
- [Azure Public IP: Basic → Standard SKU Upgrade](tradeoff-notes/azure-public-ip-basic-to-standard-upgrade.md)

## Architecture Decisions

- [ADR-0001: URL Short Code Generation](adr/0001-url-short-code-generation.md)
- [ADR-0002: URL Metadata Consistency and Replication Baseline](adr/0002-url-metadata-consistency.md)

## Author

Niels Chiu — System design、distributed systems、Azure architecture and GenAI infrastructure.
