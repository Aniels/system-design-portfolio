# System Design Portfolio

這個 repository 用來整理你的系統設計作品，作為應徵架構/系統設計相關職位時的作品集。

## 建議目錄結構

```text
.
├── case-studies/           # 每個系統設計題目的完整分析
├── architecture-diagrams/  # 架構圖（draw.io、png、svg）
├── tradeoff-notes/         # 技術選型與取捨筆記
└── templates/              # 可重用的撰寫模板
```

## 使用方式

1. 在 `case-studies/` 為每個題目建立一個子資料夾，例如：`design-url-shortener/`。
2. 將該題目的說明、需求假設、容量估算、資料模型、API 設計與可用性/一致性取捨整理成 `README.md`。
3. 把對應架構圖放在 `architecture-diagrams/`，並在 case study 內連結。
4. 將你常用的分析框架放在 `templates/`，加速後續整理。

## 下一步建議

- 先完成 1~2 個代表性題目（例如 URL Shortener、Chat System）
- 每個題目都補上「需求澄清 → 設計方案 → 取捨 → 擴充方向」
- 面試前把內容濃縮成 5 分鐘可講完的版本
