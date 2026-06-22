# RCSI 風險掃描 Prompt

> 將此 prompt 丟給 agent，替換 `{{PROJECT_PATH}}` 為目標專案路徑即可。

---

## Prompt

```
你是資料庫遷移顧問。請徹底掃描 {{PROJECT_PATH}} 專案，找出所有在啟用 SQL Server RCSI (Read Committed Snapshot Isolation) 後語意會改變、可能產生資料錯誤的程式碼模式。

### 背景知識

RCSI 啟用後，所有 READ COMMITTED 隔離級別下的 SELECT 不再取 S-lock，改為讀取「語句開始時」的已提交版本快照。
- 好處：讀寫不互擋、消除大量 deadlock
- 風險：原本靠 S-lock 阻塞「意外保護」的 read-then-write 序列失去保護，可能導致 lost update、duplicate insert、missed rows

Writers still block writers（X-lock 不變），純讀查詢和純寫操作不受影響。

### 掃描範圍

搜尋所有 .sql、.cs、.vb、.ts、.js、.aspx、.ascx、.config 檔案（排除 bin/、obj/、node_modules/、.git/）。注意 SQL 檔案可能是 UTF-8-BOM 或 UTF-16 編碼。

### 必須掃描的 7 大風險模式

請逐一掃描以下模式，每個模式報告：命中檔案數、具體檔案路徑與行號、風險等級 (CRITICAL/MEDIUM/LOW)、建議修復方案。

#### 1. NOLOCK / READ UNCOMMITTED (清理項)
- `(NOLOCK)` / `WITH (NOLOCK)` / `WITH(NOLOCK)`
- `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED`
- C# 中的 `IsolationLevel.ReadUncommitted`
- 風險：RCSI 啟用後 NOLOCK 不再需要且會掩蓋問題，應清除
- 等級：LOW（清理，非阻塞）

#### 2. 序號產生器 — SELECT → UPDATE lost update (最高風險)
- `SELECT @var = Column FROM Table` 後接 `UPDATE Table SET Column = @var + 1`
- `SELECT MAX(ID)` 用於手動產生鍵值
- `IF NOT EXISTS → INSERT / ELSE → SELECT + UPDATE` 序號計數器模式
- 關鍵字搜尋：SNStore、GetNew*No、Sequence、SeqNo、NextNo、NextVal、MaxID
- 風險：兩個併發交易讀到相同值 → 重複編號
- 等級：CRITICAL

#### 3. Check-then-act — IF [NOT] EXISTS → INSERT/UPDATE/DELETE
- `IF NOT EXISTS(SELECT ...) BEGIN INSERT ...`
- `IF EXISTS(SELECT ...) BEGIN UPDATE/DELETE ...`
- C# 中的 `FindFirst/FirstOrDefault → if null → Add` 模式
- 風險：兩個交易同時檢查同一條件，都通過 → duplicate insert 或 lost update
- 等級：CRITICAL（業務 SP）/ LOW（seed/migration script）
- 注意區分：deploy-time seed scripts vs runtime business SP

#### 4. Queue / 狀態機模式
- `SELECT TOP 1 ... WHERE Status = 'Pending'` 後接 `UPDATE Status = 'Processing'`
- Dequeue 模式：SELECT → DELETE
- 任何「認領」一筆資料然後更新狀態的模式
- 風險：兩個 worker 同時「認領」同一筆 → 重複處理
- 等級：CRITICAL（若有）/ 通常不存在於 OLTP 系統

#### 5. Watermark / ETL 增量處理
- `WHERE ModifiedDate > @lastProcessedDate`
- `WHERE ID > @lastProcessedId`
- 任何用 cursor/watermark 追蹤「上次處理到哪裡」的模式
- C# 中的 polling pattern（定時查詢新資料）
- 風險：併發未提交的 INSERT 在快照中不可見 → watermark 推進後永久漏列
- 等級：MEDIUM

#### 6. SQL Triggers
- `CREATE TRIGGER`
- 特別注意：INSTEAD OF 觸發器中的 IF EXISTS 檢查
- Trigger 內讀取同一表的其他列（在 RCSI 下讀到快照而非當前值）
- 風險：保護性檢查失效
- 等級：MEDIUM

#### 7. MERGE 語句
- `MERGE ... WHEN MATCHED ... WHEN NOT MATCHED`
- 風險：MERGE 在 snapshot isolation 下有已知 bug（可能漏失 WHEN NOT MATCHED 分支）
- 等級：MEDIUM

### 同時掃描的安全指標（正面信號）

- 已有的防禦性 locking hints：`UPDLOCK`、`HOLDLOCK`、`XLOCK`、`READPAST`、`READCOMMITTEDLOCK`、`SERIALIZABLE`
  - 區分：應用邏輯 vs DDL migration vs .NET attribute（`[System.Serializable()]` 是假陽性）
- Transaction 管理：`BEGIN TRAN`、`TransactionScope`、`BeginTransaction()`
- Isolation level 設定：`SET TRANSACTION ISOLATION LEVEL`、`IsolationLevel.`
- Concurrency tokens：`RowVersion`、`ConcurrencyCheck`、`Timestamp`

### 輸出格式

請輸出以下結構的報告：

## 摘要

| 模式 | 命中檔案數 | 風險等級 | 說明 |
|---|---|---|---|
| ... | ... | ... | ... |

## CRITICAL 發現

### [發現標題]
- **檔案**：路徑:行號
- **模式**：具體的 code pattern
- **風險**：在 RCSI 下會發生什麼
- **建議修復**：
  - 方案 A（最小改動）
  - 方案 B（最佳實踐）

## MEDIUM 發現
（同上格式）

## LOW / 清理項
（同上格式）

## 安全指標
- 已有防禦性 hints：X 處
- 無需改動的模式：列舉

## 結論
- 是否可以安全啟用 RCSI？
- 必須先修復的項目清單
- 建議的上線順序
```

---

## 使用方式

### Claude Code（背景 agent）
```
用這個 prompt，把 {{PROJECT_PATH}} 換成你的專案路徑，例如：

> 請掃描 ~/repo/my-project 的 RCSI 風險（貼上上面的 prompt）
```

### Claude Code（直接用 Agent tool）
```javascript
Agent({
  description: "RCSI risk scan",
  prompt: "（貼上 prompt，替換 {{PROJECT_PATH}}）",
  run_in_background: true
})
```

### 其他 LLM Agent
把 prompt 貼入任何有檔案存取能力的 agent（Cursor、Windsurf、Gemini CLI 等），確保 agent 有 grep/find 工具。

## 注意事項

1. **大型專案建議分階段**：先跑 grep 計數，再針對 CRITICAL 命中深入讀取
2. **假陽性**：deploy seed scripts、test data、.Designer.cs 自動生成碼中的 `[Serializable]` 都是假陽性，prompt 已提醒 agent 區分
3. **共用資料庫**：如果多個專案連同一個 DB，只需修復 SP 定義源頭（通常是一個專案管理 schema）
4. **編碼問題**：舊版 SQL Server Management Studio 匯出的 .sql 可能是 UTF-16，需注意 grep 工具是否支援
