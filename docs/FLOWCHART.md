# 任務管理系統 - 流程圖設計 (Flowchart)

這份文件描述了使用者在任務管理系統中的操作路徑 (User Flow)，以及系統背後資料處理的序列圖 (Sequence Diagram)。

## 1. 使用者流程圖 (User Flow)

展示使用者進入網站後，可以進行的各種操作與對應的頁面變化。

```mermaid
flowchart LR
    Start([使用者開啟網頁]) --> Home[首頁 - 任務清單]
    
    Home --> Action{要執行什麼操作？}
    
    Action -->|查看與篩選| Filter[點擊狀態標籤: 全部 / 未完成 / 已完成]
    Filter --> Home
    
    Action -->|新增任務| Input[在輸入框填寫任務內容]
    Input --> SubmitAdd[點擊「新增」按鈕]
    SubmitAdd -->|系統處理後更新畫面| Home
    
    Action -->|變更狀態| ClickDone[點擊任務旁的「完成」或「取消完成」]
    ClickDone -->|狀態更新後刷新| Home
    
    Action -->|刪除任務| ClickDelete[點擊任務旁的「刪除」按鈕]
    ClickDelete -->|移除項目後刷新| Home
```

## 2. 系統序列圖 (Sequence Diagram)

此處以「使用者新增任務」為例，展示從前端表單送出到後端存入 SQLite 資料庫的完整流程。

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器
    participant Flask as Flask Route
    participant Model as Task Model
    participant DB as SQLite 資料庫
    
    User->>Browser: 填寫任務名稱並點擊「新增」
    Browser->>Flask: POST /add (帶有表單資料)
    Flask->>Model: 呼叫新增任務的方法 (例如 create_task)
    Model->>DB: 執行 SQL: INSERT INTO tasks...
    DB-->>Model: 回傳執行結果 (成功)
    Model-->>Flask: 任務建立完成
    Flask-->>Browser: HTTP 302 重導向回首頁 (GET /)
    
    Browser->>Flask: GET / (重新載入首頁)
    Flask->>Model: 查詢最新任務列表
    Model->>DB: 執行 SQL: SELECT * FROM tasks
    DB-->>Model: 回傳資料庫中的任務清單
    Model-->>Flask: 回傳給控制器
    Flask->>Browser: 透過 Jinja2 渲染 index.html 並回傳
    Browser->>User: 顯示包含新任務的清單畫面
```

## 3. 功能清單對照表

將 PRD 中的主要功能對應至即將開發的 URL 路由與使用的 HTTP 方法：

| 功能名稱 | URL 路徑 | HTTP 方法 | 說明 |
| :--- | :--- | :--- | :--- |
| **顯示任務清單與篩選** | `/` | `GET` | 顯示首頁任務，並支援以 Query String (如 `/?status=completed`) 篩選。 |
| **新增任務** | `/add` | `POST` | 接收表單提交的任務標題，存入資料庫後重導向回首頁。 |
| **切換任務完成狀態** | `/toggle/<int:task_id>` | `POST` | 依據任務 ID 更新資料庫中的完成狀態 (未完成 $\leftrightarrow$ 已完成)。 |
| **刪除任務** | `/delete/<int:task_id>` | `POST` | 依據任務 ID 從資料庫中刪除該筆資料，並重導向回首頁。 |

> **備註：** 在傳統的 HTML 表單中通常只支援 GET 與 POST，因此更新 (toggle) 與刪除 (delete) 操作都會設計為 POST 請求。
