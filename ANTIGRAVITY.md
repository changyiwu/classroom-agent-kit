# classroom-agent-kit — 專案規則與指南 (ANTIGRAVITY.md)

本檔案為 AI Agent 開發與維護此專案時應遵循的專屬規則。

## 專案概要
- **專案名稱**：`classroom-tools` (ClassBuddy & classroom-cli)
- **核心功能**：網頁端課堂互動工具箱 (Vite-like / 靜態網頁) + Node.js 終端機 Google Classroom 整合工具。
- **目標平台**：Netlify (網頁部署) 及 本地 CLI 執行環境。

## 開發規則與指南

### 1. 認證與資安規範（極重要）
- **嚴禁上傳憑證**：本機產生的 `credentials.json`（GCP 桌面型憑證）與 `token.json`（OAuth 使用者 Token）已列入 `.gitignore`，請確保**絕對不可**將其強行 stage 或 commit。
- 如果需要更新 GCP OAuth 設定，請通知使用者至 Google Cloud Console 調整，而非在程式碼中寫死 API Key。

### 2. 網頁前端開發規則 (ClassBuddy)
- **UI 風格**：採用高質感、圓角友好的課堂風（以 [style.css](file:///c:/Users/chang/我的雲端硬碟/agents/antigravity/classroom-agent-kit/style.css) 中的字型 `Fredoka` 與 `Outfit` 為主）。
- **靜態主導**：維持靜態 HTML/CSS/JS 設計，非必要不引入大型重構框架，以方便直接在 Netlify 部署。
- **Google Identity Services (GIS)**：網頁端使用 GIS 彈出視窗取得 Access Token，所有 API 呼叫均在前端執行，無後端伺服器儲存 Token。

### 3. 命令列工具開發規則 (classroom-cli.js)
- **環境變數衝突**：本機執行 `gh` 命令或執行與 GitHub 整合的指令時，若環境變數存在過期的 `GITHUB_TOKEN` 會導致驗證失敗。請務必在 Powershell 中先使用 `$env:GITHUB_TOKEN=""` 清空環境變數，讓其使用系統 Keyring 儲存的有效 Token。
- **參數驗證**：任何 CLI 指令（例如 `create-assignment`）必須有嚴格的參數個數與格式檢驗，並印出清晰的 Usage 說明。

### 4. 語言與文件編寫
- 所有使用者回應、程式碼註解、說明文件與 Obsidian 駕駛艙記錄，皆統一使用**繁體中文 (zh-TW)**。
