# 潛旅平台設計總覽

**文件版本：** v0.4  
**建立日期：** 2026-05-27 ｜ **最後更新：** 2026-05-31  
**維護者：** Yun（Owner）+ 協作潛伴 + Claude（Cowork）  
**專案狀態：** 🟢 Phase 2 完成 — 全資料庫 + 網站已推上 GitHub，UX 改版為通用規劃工具

> 本文件是此 Cowork 專案的「設計記憶」。每次關閉 agent 再重開，新 agent 應先讀本文件，才能理解現況並繼續工作。同時閱讀 `README.md`（資料庫規範）和 `潛水目錄表.md`（目的地速查）。

---

## 一、專案目標

打造一個**給台灣上班族潛水愛好者使用的一站式旅遊研究平台**，幫助使用者（Yun 和潛伴）從「想去哪裡潛」到「訂好機票船宿」的完整決策流程。

### 核心 User Journey（7 步驟）

| 步驟 | 使用者需求 | 平台功能 |
|------|------|------|
| ① | 探索有哪些吸引人的潛點 | 互動地圖 + 人氣地點卡片 |
| ② | 了解各地的最佳潛季 | 潛季 × 台灣連假熱力日曆 |
| ③ | 研究哪個業者好、比較報價 | 業者比較卡（含好差評） |
| ④ | 確認目標船家的船期表 | 船期看板（手動更新 + 狀態標示） |
| ⑤ | 查機票 | Skyscanner 深連結 + 快速跳轉 |
| ⑥ | 判斷機票是否比平常貴 | 漲幅警示（比基準高 >10% 標紅） |
| ⑦ | 把方案分享給朋友 | 一鍵輸出行程方案卡 / 分享連結 |

---

## 二、技術架構

### 選型決策：全靜態方案（Phase 1）

```
Frontend（HTML / Vue 3 SPA）
  ↓ 讀取
JSON 靜態資料檔（存放在 GitHub Repo）
  ↓ 部署
GitHub Pages 或 Cloudflare Pages（免費）
  ↑ 更新資料
Claude（Cowork）手動 or 定時任務
```

**選擇理由：**
- 零伺服器成本，永久免費
- 不需在本地安裝任何軟體（JSON 是純文字，Claude 直接讀寫）
- 資料更新由 Claude 負責，不依賴任何 API key 或後端服務
- 未來若需要多人協作或即時資料，可遷移至 Supabase（前端邏輯幾乎不變）
- **多人協作**已透過 GitHub 實現：多位協作者各有自己的 AI，共同維護同一個 repo

### 多人協作架構（Phase 2 已啟用）

```
Owner（Yun）的 Claude ──┐
                         ├── GitHub repo (main) ──→ GitHub Pages
協作者甲的 AI ──────────┘
```

- **資料來源**：各自的 AI 研究並更新 Markdown + JSON
- **偏好同步**：`data/preferences.json` 記錄各協作者的潛水偏好差異
- **協作者入門**：見 `ONBOARDING.md`

### 未來可選升級路徑

| 方案 | 觸發條件 | 技術 |
|------|------|------|
| B：加入定時 fetch | 需要自動更新票價 | Cloudflare Workers（免費） |
| C：後端資料庫 | 需要多人協作編輯資料 | Supabase Free Tier |

---

## 三、檔案結構（目標狀態）

```
潛旅資料庫/（現有 Markdown 資料庫，保持不動）
├── README.md
├── PLATFORM_DESIGN.md         ← 本文件
├── 潛水目錄表.md
├── 01_個人偏好/
├── 02_印尼/ … 10_泰國/
└── …

dive-platform/（GitHub Repo 根目錄）
├── index.html                 ← 主看板（潛季指南 + 目的地列表 + 業者 + 船期 + 機票）
├── destination.html           ← ⚠️ 通用 placeholder，由 ?id=X 顯示目的地名稱（待擴充）
├── destinations/              ← 各目的地獨立頁（待建）
│   ├── kerama.html
│   ├── malapascua.html
│   ├── okinawa.html
│   ├── bohol.html
│   ├── cebu.html
│   ├── sipadan.html
│   ├── komodo.html
│   ├── raja.html
│   └── maldives.html
├── data/
│   ├── operators.json         ← 業者報價資料
│   ├── schedule_cache.json    ← 船期快取（Claude 手動更新）
│   ├── flight_index.json      ← 基準票價表
│   └── holidays_tw.json       ← 台灣連假日曆
└── .github/workflows/
    └── deploy.yml             ← 自動部署到 GitHub Pages
```

### 目的地頁面建置規範（各 destinations/X.html 建立時適用）

- **連結規範**：潛店 / 船宿名稱必須以 embedded hyperlink（`<a href="官方網站">名稱</a>`）呈現，不使用純文字
- **內容順序**：① 費用明細速查表 → ② 行程說明 → ③ 潛店/船宿評比（含好差評）→ ④ KOL 攻略卡 → ⑤ 注意事項
- **placeholder 狀態**：目前所有目的地指向 `destination.html?id=X`（通用 placeholder），各獨立頁待後續 agent 建立

---

## 四、JSON 資料 Schema

### destinations.json

```json
[
  {
    "id": "komodo",
    "name": "科莫多 Komodo",
    "country": "Indonesia",
    "flag": "🇮🇩",
    "coordinates": [8.55, 119.45],
    "costLevel": 3,
    "costRange": "NT$50,000–70,000",
    "bestMonths": [4,5,6,7,8,9,10,11],
    "avoidMonths": [12,1,2,3],
    "highlights": ["Manta Ray", "礁鯊群", "強流"],
    "minCertification": "AOW",
    "divesRequired": 0,
    "leavedays": "3–5",
    "holidayWindows": ["清明", "端午", "國慶"],
    "warnings": ["12–3月北風季，海況差"],
    "operatorIds": ["neomi-cruise", "intan-komodo"],
    "mdFile": "02_印尼/科莫多_Komodo.md"
  }
]
```

### operators.json

```json
[
  {
    "id": "neomi-cruise",
    "name": "Neomi Cruise",
    "type": "liveaboard",
    "destinations": ["komodo", "raja-ampat"],
    "pricePerNightUSD": { "lower": 450, "mid": 475, "suite": 600 },
    "typicalPackage": "6D5N Komodo",
    "typicalCostTWD": 72000,
    "rating": 4.2,
    "pros": ["2020 新船", "私密感強（12人）", "木船體驗"],
    "cons": ["木船仍有搖晃", "Wi-Fi 需額外付費"],
    "officialUrl": "https://www.neomicruise.com/",
    "contact": "+62 813 1234 6176",
    "contactMethod": "WhatsApp",
    "updatedAt": "2026-05-10"
  }
]
```

### flight_index.json

```json
{
  "updatedAt": "2026-05-27",
  "baselines": [
    {
      "origin": "TPE",
      "destination": "LBJ",
      "destinationName": "科莫多（龍目島中轉）",
      "monthlyBaseline": {
        "1": 12000, "2": 14000, "3": 11000,
        "4": 13500, "5": 11500, "6": 12000,
        "7": 15000, "8": 14500, "9": 11000,
        "10": 13000, "11": 11000, "12": 13500
      },
      "currency": "TWD",
      "alertThreshold": 0.10
    }
  ]
}
```

### holidays_tw.json

```json
{
  "note": "台灣國定假日 + 連假窗口，2026–2028，一次寫入不需更新",
  "windows": [
    {
      "name": "2027 春節",
      "publicHolidays": ["2027-01-27","2027-01-28","2027-01-29","2027-01-30","2027-01-31","2027-02-01","2027-02-02"],
      "maxTripDays": 11,
      "extraLeaveDays": 2,
      "note": "配合請 1/25–26，可達 11 天"
    }
  ]
}
```

### schedule_cache.json

```json
{
  "updatedAt": "2026-05-27",
  "vessels": [
    {
      "id": "neomi-cruise",
      "operatorId": "neomi-cruise",
      "departures": [
        {
          "date": "2027-01-27",
          "destination": "raja-ampat",
          "duration": "8D7N",
          "availableSpots": 4,
          "totalSpots": 12,
          "priceUSD": 3290,
          "status": "open",
          "sourceConfirmed": true
        }
      ]
    }
  ]
}
```

---

## 五、分階段 TODO 清單

> 狀態標示：⬜ 未開始 / 🟡 進行中 / ✅ 完成 / ⏸ 暫緩

### Phase 0：基礎準備（可在 Cowork 中完成，不需安裝任何軟體）

| # | 任務 | 狀態 | 負責 | 說明 |
|---|------|------|------|------|
| 0-1 | 撰寫本設計文件 | ✅ | Claude | 本文件 |
| 0-2 | 建立 `holidays_tw.json`（2026–2028） | ✅ | Claude | 14個連假窗口，含bestWindowByDestination對照 |
| 0-3 | 將現有 MD 轉換為 `destinations.json` | ✅ | Claude | 10個目的地，含座標/費用/季節/警示 |
| 0-4 | 將船宿 MD 轉換為 `operators.json` | ✅ | Claude | 23個業者，含warningLevel/pros/cons |
| 0-5 | 建立 `schedule_cache.json` 骨架 | ✅ | Claude | 5艘船17個船期，含pendingInquiries待詢清單 |
| 0-6 | 建立 `flight_index.json` 基準票價（第一批） | ✅ | Claude | 7條航線含月度基準＋Skyscanner深連結模板 |

### Phase 1：前端網站建立（靜態 HTML）

| # | 任務 | 狀態 | 負責 | 說明 |
|---|------|------|------|------|
| 1-1 | 現有看板已是多頁籤版（5 tabs） | ✅ | — | 硬編碼，資料完整，風格已定 |
| 1-2 | Leaflet.js 互動地圖 | ⬜ | Claude | 延後，先做高價值 JSON 驅動頁籤 |
| 1-3 | 潛季 × 假期熱力日曆 | ⬜ | Claude | 延後，現有潛水日歷頁籤已覆蓋 |
| 1-4 | 業者比較頁（篩選）| ✅ | Claude | 完成 2026-05-28 — 第 6 頁籤，讀 operators.json，含目的地/類型篩選器、warningLevel 標示、官網連結 |
| 1-5 | 機票速查 + Skyscanner 深連結 | ✅ | Claude | 完成 2026-05-28 — 第 7 頁籤，讀 flight_index.json，7 條航線月度基準票價格、旺季標示、Skyscanner 深連結 |
| 1-6 | 船期看板（含狀態顏色）| ✅ | Claude | 完成 2026-05-28 — 第 8 頁籤，讀 schedule_cache.json，5 艘船船期表、狀態顏色（open/sold_out/few_spots）、待詢問事項清單 |
| 1-7 | 行程方案輸出（列印 / 分享）| ⬜ | Claude | 下一批 |
| 1-8 | UX 重設計 — 通用規劃工具 | ✅ | Claude | 完成 2026-05-31 — 詳見下方 ADR |

**ADR 2026-05-28**：現有 5 tabs 硬編碼資料已完整，重構風險高。策略改為「新頁籤 JSON 驅動，舊頁籤保持原樣」，待新頁籤穩定後再逐步遷移。

**ADR 2026-05-31**：UX 重設計為通用多人規劃工具（見第八節 ADR 詳細記錄）。

### Phase 2：GitHub 部署 + 多人協作

| # | 任務 | 狀態 | 負責 | 說明 |
|---|------|------|------|------|
| 2-1 | Yun 在 GitHub 建立新 Repo | ✅ | Yun | https://github.com/a11ama/dive-planner |
| 2-2 | Claude 協助設定 GitHub Actions 部署腳本 | ✅ | Claude | `.github/workflows/deploy.yml` 已 push |
| 2-3 | 整個資料庫 push 到 GitHub（含 MD + JSON + HTML）| ✅ | Claude | 完成 2026-05-31，`潛旅看板.html` 改名為 `index.html`，含 MD 研究文件全部 |
| 2-4 | 確認 GitHub Pages 部署成功 | ⬜ | Yun | **Repo Settings → Pages → Source 選「GitHub Actions」後首次 push 觸發部署** |
| 2-5 | 新增 `ONBOARDING.md` + `data/preferences.json` | ✅ | Claude | 完成 2026-05-31，多人協作入門文件，含協作者偏好問卷 |
| 2-6 | 邀請協作者並設定 Collaborator 權限 | ⬜ | Yun | GitHub Repo Settings → Collaborators → Add |
| 2-7 | 公開 URL 確認並加到本文件 | ⬜ | Yun/Claude | 預計 URL：https://a11ama.github.io/dive-planner/ |
| 2-8 | UX 改版：移除固定行程橫幅，改為通用工具 | ✅ | Claude | 完成 2026-05-31，見 ADR |

### Phase 3：定時資料更新設定

| # | 任務 | 狀態 | 負責 | 說明 |
|---|------|------|------|------|
| 3-1 | 設定每月業者報價更新排程 | ⬜ | Claude（Schedule） | 每月 1 日，搜尋新報價並更新 JSON |
| 3-2 | 設定每季機票基準更新排程 | ⬜ | Claude（Schedule） | 每季首月，抓取 Skyscanner 基準票價 |
| 3-3 | 建立「船期詢問」快捷流程 | ⬜ | Claude | 你說「幫我查 XX 船 YY 月的船期」時的標準流程 |

### Phase 4：資料補全（持續進行）

| # | 任務 | 狀態 | 負責 | 說明 |
|---|------|------|------|------|
| 4-1 | 宿霧 Cebu 完整研究 | ⬜ | Claude | 目錄表標記為「高優先」 |
| 4-2 | 帛琉 Palau 研究 | ⬜ | Claude | 目錄表標記為「中優先」 |
| 4-3 | 斯米蘭詳細資料補完 | ⬜ | Claude | 已去過但資料庫無詳細檔 |
| 4-4 | 各業者船期表第一次詢問 | ⬜ | Yun | 需要 Yun 透過 WhatsApp 實際詢問或 Claude 代擬詢問訊息 |

---

## 六、部署資訊

| 項目 | 值 |
|------|------|
| 方案 | GitHub Pages（靜態） |
| Repo URL | https://github.com/a11ama/dive-planner |
| 網站 URL | https://a11ama.github.io/dive-planner/（需 Yun 在 Pages 設定啟用） |
| 資料檔路徑 | `data/`（Repo 根目錄下）|
| 最後部署時間 | 2026-05-31（UX 改版 push，Pages 啟用後可正常訪問）|
| 進入點 | `index.html`（原 `潛旅看板.html`，已改名）|

---

## 七、給下一個 Agent 的工作交接說明

如果你是接手此專案的新 Agent，請按以下順序閱讀：

1. **本文件（PLATFORM_DESIGN.md）** → 了解整體設計與開發進度
2. **ONBOARDING.md** → 多人協作快速入門（若你代表協作者，先看這裡）
3. **README.md** → 資料格式規範和研究流程
4. **潛水目錄表.md** → 現有目的地狀態（含優先級）
5. **data/preferences.json** → 潛水偏好設定（Owner Yun 的預設值）
6. **查看 Phase TODO 清單** → 找到第一個 `⬜ 未開始` 的任務繼續做

> ⚠️ 注意：`潛旅看板.html` 已改名為 `index.html`，不要建立重複的舊檔名

### 接手時的常見指令對照

| 使用者說 | Agent 應該做 |
|------|------|
| 「繼續做網站」 | 查 TODO 清單，找最近的 ⬜ 任務 |
| 「更新 XX 業者報價」 | 搜尋網頁 → 更新 operators.json → 改 updatedAt |
| 「幫我查 XX 船 YY 月船期」 | 代擬 WhatsApp 詢問訊息 OR 搜尋官網，更新 schedule_cache.json |
| 「研究 XX 新目的地」 | 照 README 的研究流程，寫 MD → 轉 JSON → 更新目錄表 |
| 「我朋友要看」 | 確認 GitHub Pages URL 是否最新，或輸出行程卡 HTML |

---

## 八、設計決策記錄（ADR）

| 日期 | 決策 | 理由 |
|------|------|------|
| 2026-05-27 | 採用全靜態方案（Phase 1）而非後端 | 零成本；資料量小（<50個目的地）；Claude 可直接讀寫 JSON 不需後端 |
| 2026-05-27 | JSON 由 Claude 產生，不在本地安裝轉換工具 | Cowork 可直接讀取 MD、寫出 JSON，對用戶零負擔 |
| 2026-05-27 | 在現有 `潛旅看板.html` 基礎上擴充，而非重建 | 保持已有的視覺風格和資料邏輯 |
| 2026-05-27 | 機票漲幅以「同月前後三個月均價基準」計算 | 避免節假日固定旺季被誤判為漲價 |
| 2026-05-28 | 新三個頁籤採用 lazy-load fetch()，舊五個頁籤保持硬編碼 | 降低重構風險；fetch() 在 GitHub Pages 正常運作；本地 file:// 開啟會顯示提示訊息而非直接崩潰 |
| 2026-05-28 | 業者比較頁使用 JS 篩選器（不用 Vue/React）| 保持零依賴、單檔案架構；業者資料量（23家）不需複雜狀態管理 |
| 2026-05-31 | 移除 plan-banner 和「2027 計畫」固定頁籤 | 原設計硬編碼春節費用資料，不適合多人通用；改為讓使用者自行進入各功能頁籤查詢 |
| 2026-05-31 | Header 改為通用標題「潛旅規劃板」 | 移除「翼潛社引薦」等特定代理商字樣，適合多人協作且分享給新朋友時不會造成困惑 |
| 2026-05-31 | 第一頁改名「潛季指南」並加入 5 步驟使用流程引導 | 新訪問者（尤其協作者朋友）不知從哪裡開始；引導卡片讓 user journey 可視化，降低上手門檻 |
| 2026-05-31 | 頁籤重排：船期看板移至機票速查之前 | 決策流程是先確認「搶得到艙位」再查機票，調整後與實際使用順序一致 |
| 2026-05-31 | 新增 ONBOARDING.md + data/preferences.json | 多人協作需要讓協作者的 AI 快速上手；preferences.json 結構化偏好資料供 agent 讀取，不依賴個人私有設定檔 |
