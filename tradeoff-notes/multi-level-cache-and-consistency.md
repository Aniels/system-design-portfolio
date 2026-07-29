# Multi-level Cache and Consistency

## 問題

大型 read-heavy 系統常同時使用 edge cache、process-local cache、regional distributed cache 與 authoritative database。多層 cache 可以降低 latency 與 origin load，但也放大 stale data、invalidation、故障降級與觀測複雜度。

## 核心原則

1. **Cache capacity 由 active working set 決定，不由 total dataset 決定。**
2. **Cache miss 不代表資料不存在。** Existence 與 uniqueness 必須由 authoritative store 判斷。
3. **Edge/local/Redis 都是 materialized copy，不是 source of truth。**
4. **先降低 mutation surface，再解 invalidation。** Immutable mapping 通常比建造複雜 global purge protocol 更可靠。
5. **Request latency 與 replication visibility lag 必須分開測。** 快速 GET 可能仍回 stale value。

## 建議層級

| 層級 | 典型內容 | 優點 | 主要風險 |
|---|---|---|---|
| Edge/CDN | globally hottest responses | 最低 latency、吸收 viral traffic | purge convergence、browser/intermediary cache |
| Process local | instance hot set | 無 network hop、成本低 | instance 間不一致、重啟即失 |
| Regional distributed cache | regional warm set | 跨 instance sharing、低 latency | cluster failure、cache stampede |
| Database | 全部資料 | authoritative correctness | 成本、tail latency、partition saturation |

## 冷熱分層策略

不必建立獨立 hot database 與 cold database。較簡單的策略是：

```text
Database retains all authoritative data
-> regional cache copies warm working set
-> local cache copies hot working set
-> edge copies hottest responses
```

以 LRU/LFU、TTL 與 access pattern 自動 promotion/eviction。是否採 LFU 應由 trace 驗證；若 traffic 呈 Zipf/power-law，LFU 通常更能保留長期熱門 key。

## Cache-aside flow

```text
L1 miss
-> L2 miss
-> authoritative point read
-> validate status/version/expiry
-> populate L2 and L1
-> return
```

不存在的 key 可使用短 TTL negative cache，抵抗 random-key scan；但必須評估新建立資料遇到 stale negative 的 global visibility 風險。

## Invalidation options

| 方案 | 優點 | 缺點 | 適用情境 |
|---|---|---|---|
| TTL only | 簡單、可靠退化 | 最壞 stale window = TTL | 可接受短暫 stale |
| Event-driven purge | 收斂較快 | event loss、out-of-order、partial purge | disable/update 需快速生效 |
| Versioned key/value | 可偵測舊版本 | metadata 與 lookup 較複雜 | 有頻繁 update |
| Tombstone | 防止舊值復活 | tombstone retention/cleanup | delete/disable correctness |
| Immutable mapping | 大幅減少 invalidation | 產品彈性較低 | short URL、content-addressed data |

通常採組合：TTL 作 correctness safety net，event-driven purge 作加速，version/tombstone 防止舊資料復活。

## Failure policy

- Local cache failure：直接 miss，無 correctness loss。
- Regional cache failure：受控 fallback database，搭配 admission control/circuit breaker，避免 DB 被瞬間打爆。
- Database unavailable：只可由仍有效且未過期的 cache serving；是否 fail-open 取決於資料風險。
- Disable/security state：通常 fail-closed 優先。
- Analytics/非關鍵 side effect：可 fail-open，避免阻塞主路徑。

## Benchmark checklist

1. 各層 hit ratio 與 marginal benefit。
2. Entry serialized size 與實際 memory overhead。
3. Cold miss P50/P95/P99。
4. Cache stampede 與 single-flight effectiveness。
5. Purge convergence P50/P99/max。
6. Redis outage 時 DB load amplification。
7. Working-set size、regional locality 與 eviction churn。
8. Stale-positive、stale-negative 的發生率與影響。
9. Equal-cost 與 equal-SLO 比較。

## 何時不該多層快取

- Dataset 很小，單層 cache 已可完整容納。
- Read traffic 不高，DB 已滿足 latency/cost。
- Mutation 很頻繁且 stale data 風險高。
- 團隊缺乏 invalidation、observability 與 incident response 能力。
- 每增加一層的 hit-rate 改善不足以抵消維運成本。
