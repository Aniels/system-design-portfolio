# ADR-0001: URL Short Code Generation

- Status: Accepted
- Date: 2026-07-29
- Owners: Niels Chiu

## Context

全球短網址服務每天建立 1 億個短網址，需跨 region 發號，並保證同一 `shortCode` 不對應到兩個不同 URL。候選方案包括 random string、UUID + Base62、auto-increment + Base62、Snowflake-like ID + Base62，以及 hash(longURL)。

## Decision Drivers

- 跨 region 唯一性。
- 不依賴 central database sequence。
- 支援約 1,157 writes/s average；10× peak 仍需有 headroom。
- 短 code。
- 降低 enumeration 與容量推測。
- 可處理 retry、clock rollback 與 worker collision。

## Considered Options

1. Random Base62 + DB collision check。
2. UUID + Base62。
3. Auto-increment ID + Base62。
4. Snowflake-like ID + reversible obfuscation + Base62。
5. Hash(longURL) + truncation。

## Decision

採用：

```text
Snowflake-like 64-bit ID
-> reversible obfuscation/permutation
-> Base62
```

DB conditional create 保留為最後的 correctness guard。Worker ID 必須以 lease 或集中註冊避免重複；clock rollback 時 worker 必須使用 logical clock、等待追平，或停止發號，不能繼續產生可能重複的 ID。

## Consequences

### Positive

- 不需每次先查 DB 才能判斷 collision。
- 可跨 region 水平擴展。
- Base62 code 相對短。
- Obfuscation 降低直接觀察時間序列與建立量的風險。

### Negative

- ID generator 本身成為需要嚴格驗證的 subsystem。
- Timestamp、region、worker、sequence bit allocation 需依實際壽命與 peak rate決定。
- Reversible obfuscation 不等同密碼學保密，仍需 rate limit 防 enumeration。

### Risks and Mitigations

- Clock rollback：logical clock、NTP alert、halt worker。
- Worker ID collision：lease/registration 與 fencing token。
- Sequence overflow：block、增加 worker 或調整 bit allocation。
- DB conditional conflict：有限次 regenerate/retry，超限回 `503`。

## Validation

- 多 region 並行發號 uniqueness test。
- Clock forward/backward chaos test。
- Worker ID lease expiry/reassignment test。
- Sequence overflow test。
- 10× create peak 的 sustained load test。
- 驗證 obfuscation 前後 code distribution 與 enumeration resistance。

## Revisit Conditions

- Random + conditional insert 在相同 SLO 下顯著更簡單且成本可接受。
- 需要使用者自訂 short code。
- Code 長度限制改變。
- 真實 create peak 超過既定 bit allocation。
