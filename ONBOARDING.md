# 協作快速入門（新 Agent / 新協作者）

> 第一次進這個 repo？按以下順序讀完，10 分鐘內可以開始貢獻。

---

## 這是什麼專案？

一套給台灣上班族打造的**潛水旅行研究平台**，幫助潛水愛好者從「想去哪裡潛」到「訂好機票船宿」的完整決策流程。

- **前端**：`index.html`（互動看板，8 個頁籤）→ 部署在 GitHub Pages
- **資料庫**：`data/*.json`（結構化資料，供前端讀取）
- **研究文件**：`02_印尼/` ～ `10_泰國/` 下的 Markdown 檔（人工研究記錄）

---

## 必讀文件

| 優先 | 檔案 | 說明 |
|------|------|------|
| ① | `README.md` | 資料格式規範、研究流程、費用定義 |
| ② | `PLATFORM_DESIGN.md` | 技術架構、開發進度（TODO 清單）、設計決策 |
| ③ | `潛水目錄表.md` | 目的地速查（含優先級與現有資料狀態） |
| ④ | `data/preferences.json` | 潛水偏好設定（Owner Yun 的預設值） |

---

## 資料架構：MD 是來源，JSON 是輸出

```
Markdown（人工研究記錄）
    ↓ Claude 轉換
data/*.json（結構化資料）
    ↓ 前端讀取
index.html（互動看板）
    ↓ GitHub Actions 自動部署
https://a11ama.github.io/dive-planner/
```

更新資料的標準流程：
1. 修改 / 新增 Markdown 研究檔
2. 同步更新對應的 JSON（`destinations.json` / `operators.json` 等）
3. Commit & push → 自動部署

---

## 多人協作模式

此專案設計為**多 agent 協作**，Owner（Yun）和熟識的潛伴各有自己的 AI，共同維護同一個資料庫。

### 如果你是協作者的 agent（非 Owner）

在開始正式任務之前，請先詢問你的 user 以下問題（照 `data/preferences.json` 的格式記錄差異）：

1. 潛水證照等級與目前潛水次數？
2. 喜歡什麼類型的潛水（大物、珊瑚礁、強流、能見度優先）？
3. 不喜歡或不接受的潛水類型？
4. 每次旅行大概預算（含機票，每人台幣）？
5. 已去過哪些潛點？
6. 容易暈船嗎？
7. 偏好幾人小船還是大型船宿？

> 由於大家是熟識的朋友，偏好差異通常很小。確認差異後，後續研究與建議可直接使用本資料庫的標準格式。

---

## 常見任務快速對照

| 你要做的事 | 做法 |
|------|------|
| 繼續開發網站 | 查 `PLATFORM_DESIGN.md` 的 TODO 清單，找最近的 ⬜ 任務 |
| 研究新目的地 | 照 `README.md` 的「研究流程」，寫 MD → 更新 JSON → 更新目錄表 |
| 更新業者報價 | 搜尋網頁 → 更新 `operators.json` → 改 `updatedAt` |
| 查或更新船期 | 更新 `schedule_cache.json`，來源須標記是否已向業者確認 |
| 把方案分享給朋友 | 確認 GitHub Pages URL：`https://a11ama.github.io/dive-planner/` |

---

## 分支策略

- `main` — 穩定版本，push 後自動部署到 GitHub Pages
- 小修改（資料更新）直接 push main
- 大功能開 feature branch，確認後 merge

---

## 資料檔說明

| 檔案 | 內容 |
|------|------|
| `data/destinations.json` | 10 個目的地（座標、費用、季節、潛點特色） |
| `data/operators.json` | 23 家業者（船型、報價、評價、官網） |
| `data/schedule_cache.json` | 船期快取（需定期人工更新或向業者詢問） |
| `data/flight_index.json` | 7 條航線月度基準票價 + Skyscanner 深連結 |
| `data/holidays_tw.json` | 台灣連假日曆 2026–2028 |
| `data/preferences.json` | 潛水偏好設定（協作者差異記錄在此） |

---

## 不在 GitHub 上的內容（.gitignore）

- `01_個人偏好/` — 含個人設定，本地保留
- `.claude/` — Claude Code 本地設定
- `.skills/` — Cowork 技能
- `*.docx` — Word 文件（用 MD 版本代替）
