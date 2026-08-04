# Google Classroom Agent & Web Sync Kit 🔗

本工具包提供 Google Classroom 的雙軌整合系統：
1. **網頁前端同步**：以純前端 Google OAuth 2.0（Google Identity Services）完成授權，支援多帳號切換而不會產生伺服器 session 衝突。
2. **終端機 CLI 控制**：一支 Node.js 命令列工具（`classroom-cli`），讓本機 AI coding agent（如 Antigravity、Claude Code、Cline 等）可以直接在對話中列出課程、匯入學生名冊、列出作業與發布公告。

---

## 📂 專案結構

```
├── index.html          # 網頁介面，內含 Google Classroom 登入彈出視窗
├── app.js              # 前端 OAuth 流程與 Classroom REST API 呼叫邏輯
├── style.css           # Classroom 面板的樣式規範
├── classroom-cli.js    # 本機 API 操作的 CLI 腳本
└── package.json        # Node.js 相依套件與 bin 路徑對應
```

---

## ⚙️ Google Cloud Platform 設定（初次安裝）

不論要使用網頁介面或 CLI 指令，都必須先在 [Google Cloud Console](https://console.cloud.google.com/) 設定憑證：

### 1. 啟用 Classroom API
- 在專案搜尋列輸入 `Google Classroom API`，點選 **啟用**。
- 前往 `OAuth 同意畫面`（OAuth Consent Screen）→ 選擇 **外部（External）** → 在 **測試使用者（Test Users）** 加入你要測試的 Google 帳號（開發者狀態下相當重要）。
- 確認已加入 `classroom.courses.readonly`、`classroom.rosters.readonly` 與 `classroom.announcements` 等範圍（作業相關則需 `classroom.coursework.me` / `classroom.coursework.students`）。

### 2. 建立 OAuth 憑證

#### A. 網頁應用程式 Client ID（供瀏覽器同步使用）
- 前往 `憑證` → **建立憑證** → **OAuth 用戶端 ID** → 選擇 **網頁應用程式**。
- 在 **已授權的 JavaScript 來源** 加入你的本機位址（`http://localhost:5500`）與正式的 Netlify 位址（`https://your-app.netlify.app`）。
- 複製產生的網頁版 Client ID，貼到網頁介面中即可同步。

#### B. 桌面應用程式 Client ID（供 CLI 控制使用）
- 前往 `憑證` → **建立憑證** → **OAuth 用戶端 ID** → 選擇 **桌面應用程式**。
- 記下 client ID，並下載產生的憑證 JSON 檔。
- 將下載的 JSON 檔改名為 **`credentials.json`**，存放在與 `classroom-cli.js` 相同的目錄下。

---

## 💻 網頁端 OAuth 流程整合（`app.js`）

網頁端整合新版 **Google Identity Services（GIS）**，直接在瀏覽器內請求授權 token，可避免老師同時登入多個 Google 帳號時發生的瀏覽器 session 衝突。

```javascript
let googleAccessToken = null;

// 初始化 GIS Token client
const tokenClient = google.accounts.oauth2.initTokenClient({
  client_id: "YOUR_GOOGLE_CLIENT_ID",
  scope: "https://www.googleapis.com/auth/classroom.courses.readonly https://www.googleapis.com/auth/classroom.rosters.readonly",
  callback: (tokenResponse) => {
    if (tokenResponse && tokenResponse.access_token) {
      googleAccessToken = tokenResponse.access_token;
      // 之後即可用 Authorization 標頭呼叫各個 API 端點
    }
  }
});

// 觸發登入彈出視窗
tokenClient.requestAccessToken({ prompt: "consent" });
```

---

## ⌨️ CLI 工具安裝與使用

你可以把腳本連結成全域指令，讓終端機或 coding agent 在任何位置都能執行。

### 1. 安裝
安裝 Node.js 相依套件：
```bash
npm install
```

### 2. 建立全域連結
在本機建立全域 symlink：
```bash
npm link
```

### 3. CLI 指令
直接在終端機執行下列指令；若使用 AI Agent，Agent 也能代你呼叫這些指令來查詢資訊或寫入更新：

* **授權（只需執行一次）**：
  ```bash
  classroom-cli auth
  ```
  *（會開啟瀏覽器要求登入，並把 OAuth access token 存入 `token.json`）*

* **列出進行中的課程**：
  ```bash
  classroom-cli list-courses
  ```

* **列出課程學生名冊**：
  ```bash
  classroom-cli list-students <courseId>
  ```

* **建立課程作業**：
  ```bash
  classroom-cli create-assignment <courseId> "作業標題" "作業說明／描述"
  ```

* **發布訊息串公告**：
  ```bash
  classroom-cli post-announcement <courseId> "公告內容"
  ```

* **列出課程作業（Coursework）**：
  ```bash
  classroom-cli list-coursework <courseId>
  ```

---

## 🩺 安裝疑難排解（Troubleshooting）

依「最常踩」排序，安裝／首次授權時若卡住先看這裡：

### 1. CLI 憑證一定要選「桌面應用程式」類型
`classroom-cli` 透過 `@google-cloud/local-auth` 授權，**只吃 Desktop application 類型的 OAuth 憑證**。
若誤用 Web application 那組，授權會失敗或拿不到 refresh token。下載後務必改名為 **`credentials.json`**，
放在與 `classroom-cli.js` **同一層目錄**（程式以 `__dirname` 定位，放錯就會報 `credentials.json is missing`）。
> 注意：CLI（Desktop）與網頁端（Web App）用的是**兩組不同的 Client ID**，不要互換。

### 2. OAuth 同意畫面在「測試中」→ 必須把自己加進 Test Users
同意畫面選 External 後，**要把你要登入的 Google 帳號加到 Test Users 名單**，否則 `classroom-cli auth`
會被擋（`access_denied` / 未通過驗證）。使用學校 Google Workspace 帳號時，若網域管理員鎖定第三方
App 存取，個人將無法授權，需請管理員開放或改用一般 Gmail 測試。

### 3. 改了 Scope 之後，務必刪掉舊的 `token.json` 再重新授權
本 CLI 需要以下 5 個 scope（缺了會在跑 `post-announcement` / `create-assignment` 時噴 403）：
`classroom.courses.readonly`、`classroom.rosters.readonly`、`classroom.announcements`、
`classroom.coursework.me`、`classroom.coursework.students`。
程式只要偵測到 `token.json` 存在就會沿用舊權限，**改 scope 後不會自動更新**，
因此調整 scope 後請：
```bash
# 刪掉舊 token，再重新授權
rm token.json   # Windows PowerShell: Remove-Item token.json
classroom-cli auth
```

### 4. `npm link` 失敗或 `classroom-cli` 找不到指令
`npm link` 需寫入全域 node_modules，Windows 上有時需系統管理員權限；若全域 npm bin 不在 PATH，
打 `classroom-cli` 會 command not found。**最簡單的退路是不 link，直接用：**
```bash
node classroom-cli.js list-courses
```
效果完全相同。

### 5. Node.js 版本
`googleapis@^173` 建議在 **Node.js 18 / 20 LTS（含）以上**執行，Node 太舊可能在 `npm install` 階段失敗。

### 6. 找不到 courseId
`list-students`、`post-announcement` 等指令的 `<courseId>` 是**數字 ID**，不是課程名稱。
先跑 `classroom-cli list-courses`，從輸出的 `id` 欄位複製對應課程的 ID。

---

## 🚀 進階功能：讓 AI Agent 下載與讀取學生作業檔案

若要讓 AI Agent 具備「下載與讀取學生所上傳的作業檔案（如 Google 文件、PDF、Word 檔）」之功能，必須額外設定 Google Drive API 權限：

### 1. 啟用 Google Drive API
* 前往 [Google Cloud Console API 庫](https://console.cloud.google.com/apis/library)，搜尋並啟用 **`Google Drive API`**。

### 2. 新增 OAuth Scopes 範圍
* 進入「Google Auth Platform」->「**資料存取權** (Scopes)」。
* 點擊「**新增或移除範圍**」，搜尋並勾選以下權限：
  * `https://www.googleapis.com/auth/drive.readonly` (查看您 Google 雲端硬碟中的檔案)
* 點選「**更新**」，並點擊「**儲存**」。

### 3. 重置本地認證金鑰
* 由於 Scopes 被修改，舊的金鑰將無法使用。請刪除本地的 `token.json`：
  ```bash
  # Windows PowerShell
  Remove-Item token.json
  # macOS / Linux
  rm token.json
  ```
* 重新執行認證指令，並在瀏覽器授權畫面中，勾選新增的「查看您的 Google 雲端硬碟檔案」權限：
  ```bash
  node classroom-cli.js auth
  ```
