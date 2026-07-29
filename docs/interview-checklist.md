# System Design Interview Checklist

## Requirements

- [ ] 釐清使用者、核心 use cases 與 out-of-scope
- [ ] 定義讀寫比例、流量、資料量與地理範圍
- [ ] 明確寫出 availability、latency、durability、consistency SLO

## Architecture

- [ ] 先提出最小可行架構，再逐步擴充
- [ ] 說明 sync 與 async 邊界
- [ ] 指出 stateful component 與 source of truth
- [ ] 標示 cache、queue、database 與 object storage 的責任

## Data

- [ ] 定義資料模型、index 與 partition key
- [ ] 處理 hot key、hot partition 與 rebalancing
- [ ] 說明 replication、backup、retention 與 deletion

## Correctness

- [ ] 說明 ordering、delivery semantics 與 idempotency
- [ ] 區分 strong consistency 與 eventual consistency
- [ ] 處理 duplicate、late、out-of-order 與 missing events

## Reliability

- [ ] 定義 timeout、retry、backoff with jitter、circuit breaker
- [ ] 說明 dependency failure 與 degraded mode
- [ ] 定義 fail-open / fail-closed 決策
- [ ] 說明 region outage、RPO、RTO 與 failback

## Operations

- [ ] 定義 SLI、SLO、alert 與 error budget
- [ ] 說明容量測試、壓力測試、soak test 與 chaos test
- [ ] 評估成本、維運複雜度與團隊能力

## Communication

- [ ] 每項重大選擇都提出至少一個替代方案
- [ ] 不把產品名稱當成設計理由
- [ ] 清楚指出最大瓶頸與下一個演進點
- [ ] 能在五分鐘內摘要整體設計
