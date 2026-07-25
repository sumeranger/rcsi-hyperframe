# 用故事理解 Read Committed Snapshot Isolation (RCSI)

使用 **Hyperframe** 製作的技術動畫影片，透過故事化方式介紹 SQL Server 中的 **Read Committed Snapshot Isolation (RCSI)** 機制。

影片將帶你理解：

- 為什麼傳統 Read Committed 會產生 Lock Blocking
- RCSI 如何透過 Row Versioning 解決讀取阻塞
- Transaction 與 Snapshot Version Store 如何協作
- 「讀者不阻塞寫者、寫者不阻塞讀者」背後的運作原理

🎬 影片：
1. 下載專案後, 執行以下指令產出
```
bun run render:detail
```

---

## 專案目的

資料庫 Isolation Level 通常是後端工程師最容易只記語法、卻難以真正理解的主題。

本專案嘗試使用動畫與故事場景，將 RCSI 的核心概念視覺化：

> 不再只是記住 `ALTER DATABASE SET READ_COMMITTED_SNAPSHOT ON`，而是真正理解 SQL Server 背後如何管理資料版本。

---

## 製作工具

本影片使用：

- 🎞️ **Hyperframe** — AI 輔助動畫影片製作
- 📝 Script 驅動影片內容
- 🎨 視覺化呈現 Database Transaction Flow

