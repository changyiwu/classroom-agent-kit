# classroom-agent-kit（專案藍圖）

> 本檔為跨 Agent 通用的專案藍圖（AGENTS.md 開放標準）。任何 Agent 的每個 session 都應先讀本檔＋`handoff.md`。

## 專案簡介

兩塊核心內容：ClassBuddy 網頁端課堂互動工具箱，以及 `classroom-cli.js` Google Classroom 命令列工具。目標平台為 Netlify 靜態網站與本地 Node.js CLI 執行環境。

## 關鍵時程

<!-- 目前無固定時程 -->

## 目標與路線圖

- [x] 階段一：ClassBuddy 網頁端與 `classroom-cli.js` CLI 成形
- [x] 階段二：專案規則完整遷移為跨 Agent `agents.md`，移除舊規則檔
- [ ] 階段三：以手機或平板驗證 ClassBuddy 與 Google Classroom 名冊同步
- [ ] 階段四：後續修改 CLI 時維持嚴格參數驗證與 Usage 說明

## 資料夾結構

```
classroom-agent-kit/
├─ index.html            # ClassBuddy 網頁入口
├─ app.js                # 網頁端邏輯
├─ style.css             # Fredoka／Outfit 字型的課堂風格樣式
├─ classroom-cli.js      # Google Classroom 命令列工具
├─ package.json  package-lock.json
├─ README.md
├─ agents.md             # 本檔：專案藍圖
├─ handoff.md            # 交接檔（每次收工必更新）
├─ scratch/              # 本機暫存
├─ .netlify/             # Netlify 部署設定
└─ .gitignore            # 已排除 credentials.json / token.json
```

> `credentials.json`（GCP 桌面型憑證）與 `token.json`（OAuth 使用者 Token）存在於本機工作目錄，**受 `.gitignore` 保護，絕不可提交**。

## 同步層級（本專案初始化至第 3 層級）

| 層級 | 平台 | 位置 | 讀取時機 |
|------|------|------|---------|
| L1 | 本地（GDrive） | `agents.md`＋`handoff.md` | 每個 session |
| L2 | GitHub | https://github.com/changyiwu/classroom-agent-kit （公開） | 指定時 |
| L3 | Obsidian | `classroom-agent-kit/專案工作流程.md` | 有需要時 |

## 三個檔案的職責（依「時效性」分家，不是依「詳細程度」）

| 檔案 | 時效 | 寫入方式 | 放什麼 |
|------|------|---------|--------|
| `handoff.md` | **只對下一個 session 有效**，過期即丟 | 每次收工整份重寫 | 做到哪、下一步、**這次**的暫時 workaround |
| `agents.md`（本檔） | **長期有效**，每個 session 都適用 | 只有規則本身變了才改 | 目標、路線圖、常設規則、結構 |
| Obsidian／`git log` | **歷史**：發生過什麼、為什麼 | 只增不刪 | 決策紀錄、踩坑完整版、逐次進度 |

驗收標準：**`handoff.md` 整份刪掉，不應損失任何長期資訊**——會的話代表該升級進本檔卻沒升級。

**本檔不要出現的東西**：❌ `## 最近進度`／逐次工作紀錄、❌ 決策理由與踩坑完整版。2026-08-03 移除了 `## 最近進度`，內容逐條比對後已在 L3 筆記的〈🗓️ 最近更動紀錄〉——**是主動移除，不是遺漏，不要補回來**。踩過的坑只把**結論**收斂成一條祈使句寫進〈工作約定〉，原因留 L3。

## 工作約定

- 任何 Agent、任何電腦：**開工先讀 `handoff.md`，收工必更新 `handoff.md`**
- 修改共用檔案前先讀最新內容，避免覆蓋其他 Agent 的變更
- 所有使用者回應、程式碼註解、說明文件與 Obsidian 專案紀錄使用繁體中文（zh-TW）
- 涉及檔案操作時回報完整產出位置；Windows 指令優先使用 PowerShell
- 提交前檢查 staged diff，避免把其他工作或敏感資料一起納入

## 認證與資安規範

- **嚴禁上傳** 本機產生的 `credentials.json` 與 `token.json`。兩者即使已列入 `.gitignore`，也不可強制 stage 或 commit
- 更新 GCP OAuth 設定時，引導使用者前往 Google Cloud Console 調整，不要把 API key、OAuth token 或憑證寫死在程式碼中
- 執行 `gh` 或 GitHub 整合指令時，若過期的 `GITHUB_TOKEN` 造成驗證失敗，先在目前 PowerShell session 執行 `$env:GITHUB_TOKEN=""`，讓工具改用系統 Keyring 中的有效憑證

## 網頁前端規則（ClassBuddy）

- UI 維持高質感、圓角友好的課堂風格，以 `style.css` 使用的 `Fredoka` 與 `Outfit` 字型為基礎
- 維持靜態 HTML、CSS、JavaScript 架構；非必要不引入大型前端框架，以利直接部署至 Netlify
- 網頁端使用 Google Identity Services 彈出視窗取得 Access Token；API 呼叫在前端執行，不由後端伺服器儲存 Token

## 命令列工具規則（classroom-cli.js）

- 每個 CLI 指令（例如 `create-assignment`）都必須嚴格檢查參數數量與格式，並提供清楚的 Usage 說明
