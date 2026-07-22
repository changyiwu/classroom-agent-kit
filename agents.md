# classroom-agent-kit（跨 Agent 專案規則）

> 本檔供不同 Agent 在開發與維護本專案時共同遵循。

## 專案概要

- 專案名稱：`classroom-agent-kit`
- 核心內容：ClassBuddy 網頁端課堂互動工具箱，以及 `classroom-cli.js` Google Classroom 命令列工具。
- 目標平台：Netlify 靜態網站與本地 Node.js CLI 執行環境。
- Obsidian 筆記：`classroom-agent-kit/專案工作流程.md`

## 認證與資安規範

- 嚴禁上傳本機產生的 `credentials.json`（GCP 桌面型憑證）與 `token.json`（OAuth 使用者 Token）。兩者即使已列入 `.gitignore`，也不可強制 stage 或 commit。
- 更新 GCP OAuth 設定時，引導使用者前往 Google Cloud Console 調整，不要把 API key、OAuth token 或憑證寫死在程式碼中。
- 提交前檢查 staged diff，避免把其他工作或敏感資料一起納入。

## 網頁前端規則（ClassBuddy）

- UI 維持高質感、圓角友好的課堂風格，以 `style.css` 使用的 `Fredoka` 與 `Outfit` 字型為基礎。
- 維持靜態 HTML、CSS、JavaScript 架構；非必要不引入大型前端框架，以利直接部署至 Netlify。
- 網頁端使用 Google Identity Services 彈出視窗取得 Access Token；API 呼叫在前端執行，不由後端伺服器儲存 Token。

## 命令列工具規則（classroom-cli.js）

- 執行 `gh` 或 GitHub 整合指令時，若過期的 `GITHUB_TOKEN` 造成驗證失敗，先在目前 PowerShell session 執行 `$env:GITHUB_TOKEN=""`，讓工具改用系統 Keyring 中的有效憑證。
- 每個 CLI 指令（例如 `create-assignment`）都必須嚴格檢查參數數量與格式，並提供清楚的 Usage 說明。

## 語言與文件

- 所有使用者回應、程式碼註解、說明文件與 Obsidian 專案紀錄使用繁體中文（zh-TW）。
- 涉及檔案操作時回報完整產出位置；Windows 指令優先使用 PowerShell。

## 最近進度

- 2026-07-22：將 Antigravity 專用規則完整遷移為跨 Agent `agents.md`，並移除舊規則檔。
