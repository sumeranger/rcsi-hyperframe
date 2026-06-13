# SQL Server RCSI 研究報告(影片素材原始來源)

> 影片 `rcsi-brief.mp4` 的所有技術主張依據。關鍵事實均經 Microsoft Learn 與 SQL Server 社群權威(Paul White、Kendra Little、Brent Ozar、Erik Darling、Michael J. Swart、Dmitri Korotkevitch)交叉查證。

## 金句

- **Brent Ozar**:Readers don't block writers, writers don't block readers — but writers still block writers.
- **Erik Darling**:大家都嘲笑滿天飛的 NOLOCK,但沒人真正解決逼出 NOLOCK 的阻塞問題。RCSI 就是正解。
- **最強商業論點**:Azure SQL Database 與 Microsoft Fabric 預設就開 RCSI——我們只是把地端調成微軟眼中「現代的預設」。

## 機制

- 預設 READ COMMITTED(locking):SELECT 逐列取 S 鎖,寫入取 X 鎖持有到交易結束;S/X 不相容 → 讀寫互擋。Paul White 指出 locking RC 連語句層一致性都不保證(掃描可漏列/重複讀)。
- RCSI:寫入者把舊版複製到 tempdb version store,列尾加 14-byte 指標(6B XSN + 8B RID);SELECT 不取 S 鎖(只 Sch-S),讀「語句開始時」最新已提交版本(statement-level snapshot)。
- RCSI vs SNAPSHOT:RCSI 語句級、零改 code、無 update conflict(error 3960 僅 SNAPSHOT 會發生);SNAPSHOT 交易級、需改 code + retry。
- 寫 vs 寫不變:X 鎖照常排隊,資料正確性不打折。
- 開啟:`ALTER DATABASE [DB] SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;`(短暫獨佔、數秒、可逆)。

## 本案:SELECT–DELETE key lookup deadlock

1. SELECT 在 NC index 取得 S 鎖
2. DELETE 在 clustered index 取得 X 鎖
3. SELECT key lookup → 等 CI 上的 X
4. DELETE 維護 NC index → 等 NCI 上的 S → 成環
5. deadlock monitor(~5 秒)選 rollback 成本低者砍掉 → error 1205

兩條各自合理的語句、毫秒級時間差即可觸發——逐案修索引是打地鼠。RCSI 讓 SELECT 完全不取 S 鎖,環的兩條邊都拆掉:**結構性消失,不是機率變低**。

## 風險與對策

| 代價 | 對策 |
|---|---|
| tempdb version store 壓力;長交易使其膨脹 | `sys.dm_tran_version_store_space_usage`、`sys.dm_tran_top_version_generators`、Longest Transaction Running Time 告警;SQL 2019+ ADR 將版本移入使用者 DB(PVS,約 +10–15% 空間) |
| 每列 +14 bytes(首次修改時加)、可能 page split | staging 壓測;多數 OLTP 無感 |
| 語意改變:讀到語句開始時快照(稍舊資料) | 盤點 check-then-act / queue / watermark ETL;局部加 `UPDLOCK`(+`READPAST`)、`READCOMMITTEDLOCK` |

經典案例:Craig Freedman「Read Committed and Updates」;Kendra Little lost updates(2023);Michael J. Swart watermark ETL 漏列(2023)。

## 上線步驟

1. 盤點 codebase:NOLOCK、queue pattern、SELECT-then-UPDATE、watermark ETL
2. 測試環境開啟 + 負載重放,觀察 tempdb/CPU/正確性
3. 建立 version store / tempdb / 長交易監控
4. 修補風險查詢(UPDLOCK / READCOMMITTEDLOCK)
5. 離峰維護窗口執行 ALTER
6. 觀察 1–2 週(deadlock 應驟降),保留 OFF 回退程序

## 主要來源

- Microsoft Learn: Transaction locking and row versioning guide / SET TRANSACTION ISOLATION LEVEL / Optimized locking / ADR
- Paul White: The Read Committed Isolation Level; RCSI; Data Modifications under RCSI; The SNAPSHOT Isolation Level; The Read Uncommitted Isolation Level (sql.kiwi / sqlperformance.com)
- Kendra Little: Implementing Snapshot or RCSI — A Guide; How to Choose Between RCSI and Snapshot; Lost Updates under RCSI; "Read Committed is Bonkers"
- Brent Ozar: RCSI Writers Block Writers; Can SELECTs Win Deadlocks?
- Erik Darling: Should An Optimistic Isolation Level Be The New Default?
- Michael J. Swart: Watch Out For This Use Case When Using RCSI
- Dmitri Korotkevitch: Key Lookup Deadlock
