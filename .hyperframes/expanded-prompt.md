# Expanded Production Prompt — 「凌晨三點的犧牲者」: 說服主管開啟 RCSI

## Title + Style

- 片名:**凌晨三點的犧牲者 — 為什麼我們該開啟 RCSI**
- 規格:1920×1080,~112 秒,9 場景,無旁白(文字驅動,可靜音觀看)
- 風格依 `design.md`「Lock & Light」:bg `#0B0E14`、panel `#121826`、fg `#E8ECF4`、dim `#8B94A7`、
  alarm `#E5484D`(僅鎖/死結)、go `#3DD68C`(僅 RCSI/通行)、caution `#F0B429`(僅風險)、steel `#4A5878`(結構)
- 字型:標題 `Noto Serif CJK TC 900`、內文 `Noto Sans CJK TC 350`、code/數據 `JetBrains Mono`(中文 fallback `Noto Sans Mono CJK TC`)

## Rhythm 宣言

**HOOK(紅色警報)- build(機制圖解)- PUNCH(死結成環)- HARD-CUT-SOLVE(綠色一行指令)- flow(版本機制)- proof(效益與證據)- honest(琥珀風險)- plan(行動)- resolve(金句收尾)**

能量曲線:high → medium → HIGH PEAK → 落差(靜→爆)→ medium-calm → medium → medium(誠實沉穩)→ medium → calm。

## Global Rules

- 主轉場:**push slide / blur-through(medium, 0.4-0.5s, power2/power3)**;
  危機→解方(S3→S4)用 **hard cut + 全畫面色彩語言反轉**(紅世界 → 暗場綠光),是全片最大的修辭轉折。
- 每場景 8-10 元素:BG(radial glow、grid、ghost 鎖/分支圖形、grain)+ MG(內容)+ FG(monospace 元資料、hairline、tick)。
- 所有裝飾帶 ambient motion(breath/drift/pulse),掛在 tl 上(非裸 gsap.to)。
- 入場一律 `fromTo`;轉場即出場(exit 動畫僅最終場景)。
- 中文行高 ≥1.55,數字 tabular-nums,SQL 一律 mono。
- 場景間色彩語意絕不混用:紅=鎖;綠=RCSI;琥珀=風險。

## Per-Scene Beats(時間軸)

### S1 — HOOK:深夜告警(0–12s)

- **Concept**:凌晨 3:17 的監控室。黑暗中只有警報在閃。真實的 SQL Server error 1205 訊息逐字打出——「Transaction (Process ID 52) was deadlocked … and has been chosen as the deadlock victim.」觀眾感受:這不是假設,這是我們上週的生產事故。
- **Mood**:NOC 監控室、事故 war-room。冷,然後刺紅。
- **Depth layers**:BG = 深藍黑 + 紅色 radial glow(呼吸)+ 微格線;MG = 終端機卡片(error 訊息 types on)+ 大標「又一筆交易,被資料庫親手砍掉」;FG = mono 時間戳 `03:17:42`、`spid 52`、`error 1205` 標籤、hairline 紅線。
- **Choreography**:時間戳 types on(低能量)→ 終端卡片 SLIDES 入(power3.out)→ error 文字逐字打出 → 「deadlock victim」一詞被紅色 highlight STAMPS → 大標 SLAMS(expo.out)→ 紅 glow PULSES。
- **Transition out**:blur-through 0.45s(進入講解模式)。

### S2 — 機制:預設世界的鎖(12–28s)

- **Concept**:拉開引擎蓋。預設 READ COMMITTED 隔離級別下,SELECT 要拿 shared lock(S),DELETE 要拿 exclusive lock(X),S 與 X 互不相容——讀的人和寫的人在同一張桌子前互相卡位。
- **Mood**:工程藍圖、教學圖解。精準、冷靜。
- **Depth layers**:BG = 格線 + steel ghost 表格圖形;MG = 一張「資料表」橫條 + 左右兩個 process 膠囊(SELECT 藍白 / DELETE 紅)+ 鎖圖示;FG = 相容性矩陣小卡(S×X = ✗)、mono 標籤 `S lock` `X lock`、解說字幕。
- **Choreography**:表格 DRAWS(scaleX 0→1)→ SELECT 膠囊 SLIDES 自左、DELETE 膠囊 SLIDES 自右(不同 ease)→ 各自的鎖圖示 DROPS 落在表格上 → 相容矩陣 FLIPS 入,✗ STAMPS 紅色 → 字幕 fades。
- **Transition out**:push slide 0.4s。

### S3 — PUNCH:死結成環,選一個殺(28–46s)

- **Concept**:本片危機頂點。重演真實事故:SELECT(走 NC index → key lookup)與 DELETE(走 clustered index → 回頭碰 NC index)以**相反順序**取鎖。兩個等待箭頭接成一個環——wait graph 成環即死結。SQL Server 的死結偵測器(每 5 秒巡邏)發現環,挑 rollback 成本低的那個,當場砍掉。紅色大字:**犧牲者**。
- **Mood**:慢動作車禍重建 + 法醫報告。讓主管「看懂」那份 deadlock graph。
- **Depth layers**:BG = 紅 glow 增強(心跳式 pulse)+ 格線;MG = 兩個資源節點(`Clustered Index` / `Nonclustered Index`)+ 兩個 process 節點 + 4 條依序點亮的箭頭;FG = mono 步驟編號 ①②③④、`wait chain` 標籤、偵測器掃描線。
- **Choreography**:節點 CASCADE 入 → 箭頭依序 DRAWS(每步 0.8s,壓迫感遞增)→ 第 4 條箭頭接上瞬間,整個環 FLASHES 紅 + 畫面微震(x ±6px)→ 掃描線 SWEEPS → SELECT 節點被紅 X SHATTERS → 「犧牲者」二字 SLAMS(serif 900, 130px)→ 底部 mono:`error 1205 · rollback cost 較低者死`。
- **Transition out**:**hard cut**(0s)→ S4 全黑靜場。全片最重的一刀。

### S4 — SOLVE:一行指令(46–58s)

- **Concept**:落差。黑場 0.8 秒,只有一行綠色 SQL 緩緩 type on:`ALTER DATABASE [DB] SET READ_COMMITTED_SNAPSHOT ON;`。然後大標:「讓讀的人,不再拿鎖。」世界從紅翻綠。
- **Mood**:救贖時刻、開燈瞬間。極簡,讓一行字扛全場。
- **Depth layers**:BG = 黑 → 綠 radial glow 緩亮;MG = SQL 終端卡 + 大標;FG = mono 註記 `不需改一行 application code`、`WITH ROLLBACK IMMEDIATE 數秒完成切換`、綠 hairline。
- **Choreography**:SQL types on(打字機,逐字)→ 游標 BLINKS → 執行音效感:整行 FLASHES 綠 → 大標 RISES(power3.out, 慢 0.9s)→ 註記 fades 入。
- **Transition out**:blur crossfade 0.5s(柔和,進入講解)。

### S5 — 機制:版本快照如何運作(58–76s)

- **Concept**:RCSI 的引擎室。寫入者更新列時,舊版本被複製到 tempdb 的 version store,列上掛 14-byte 指標;讀取者不拿 shared lock,直接讀「語句開始時已提交」的版本。比喻:**寫的人改實體,讀的人看快照** ——同一張表,兩條動線,互不相撞。
- **Mood**:精密機械剖面圖、地鐵雙軌動線圖。
- **Depth layers**:BG = steel 網格 + 綠 glow;MG = 中央資料列卡片 + 右側 tempdb version store 面板 + 左側 SELECT/右側 DELETE 雙動線箭頭;FG = `14-byte pointer` mono 標籤、版本鏈節點、`statement-level snapshot` 標籤、字幕。
- **Choreography**:資料列卡 SLIDES 入 → DELETE 動線 DRAWS 紅(碰實體列)→ 舊版本卡 PEELS OFF 滑進 version store(綠)→ 指標鏈 DRAWS → SELECT 動線 DRAWS 綠(繞過鎖、直達快照版本)→ 兩動線同時 PULSES:互不相撞 → 字幕「讀寫從此各走各的軌道」fades。
- **Transition out**:push slide 0.4s。

### S6 — 效益與證據(76–90s)

- **Concept**:給主管的證據清單。四張效益卡 + 一個權威證據:**Azure SQL Database 預設就開著 RCSI**——微軟自己的雲,出廠值就是這個。對照組:NOLOCK 是讀髒資料的毒藥,RCSI 才是正解。
- **Mood**:法庭呈堂證物、盡職調查報告。
- **Depth layers**:BG = 綠 glow + 格線;MG = 效益卡(讀不阻塞寫、寫不阻塞讀 / reader-writer deadlock 直接消失 / 0 行程式碼修改 / 取代危險的 NOLOCK)staggered;FG = Azure 證據橫幅(mono 引用)、✓ tick 動畫、hairline。
- **Choreography**:效益卡 CASCADE(交錯方向入場)→ 每卡 ✓ DRAWS → Azure 橫幅 SLIDES UP 壓底(back.out 輕微 overshoot)→ NOLOCK 對照小卡 STAMPS 紅色刪除線。
- **Transition out**:vertical push 0.45s(換氣,進入誠實段落)。

### S7 — 風險:誠實的代價清單(90–102s)

- **Concept**:語氣轉折:「天下沒有免費的午餐。」琥珀色。三項代價:① tempdb 變重(version store 需要空間與監控,長交易會讓它膨脹);② 每列 14-byte 開銷;③ 語意改變——讀到的是「語句開始那一刻」的已提交資料,依賴「SELECT 會等寫鎖放掉」行為的程式(先查再改、用查詢當佇列)需要逐一盤點,必要處加 UPDLOCK。每項代價旁邊直接給「對策」。
- **Mood**:工程風險評估會議。冷靜、對等、不粉飾。
- **Depth layers**:BG = 琥珀 glow(低)+ 格線;MG = 三張「代價 ↔ 對策」對開卡;FG = mono 監控指令 `sys.dm_tran_version_store_space_usage`、`tempdb` 標籤、量表小圖。
- **Choreography**:標題「代價,說清楚」FADES(沉穩,慢)→ 三卡依序 UNFOLDS(每張 0.5s)→ 每張對策列 types on → tempdb 量表 FILLS 琥珀。
- **Transition out**:blur crossfade 0.5s。

### S8 — 行動方案(102–112s)+ 金句收尾

- **Concept**:不是「現在就開」,而是一個負責任的三步上線計畫:① 測試環境開啟 + 壓測 tempdb;② 盤點依賴鎖行為的程式碼;③ 離峰維護窗口執行切換 + 監控 version store。最後畫面安靜下來,金句:**「與其讓資料庫每天選一個犧牲者,不如讓讀與寫,各走各的路。」** 落款 RCSI 綠色字樣,淡出。
- **Mood**:會議結論頁 → 電影結尾黑底字卡。
- **Depth layers**:BG = 綠 glow 漸暗;MG = 三步驟時間軸(編號圓點 + 連接線 DRAWS)→ 切至金句字卡;FG = mono 標籤 `staging → audit → cutover`、hairline。
- **Choreography**:步驟圓點 POPS 依序(back.out)→ 連接線 DRAWS → 步驟文字 SLIDES → 全部 fades(最終場景允許出場)→ 金句兩行 RISES 緩慢(1.2s, sine)→ 「RCSI」綠字 BREATHES → 全黑。
- **Transition**:此為最終場景,fade to black 收尾。

## Recurring Motifs

- 鎖圖示(S1–S3 紅鎖 → S4 之後鎖被「放下」消失)
- monospace 時間戳/標籤貫穿全片(機器的聲音)
- 格線背景全片一致;glow 顏色 = 該幕語意色
- 「兩條動線」圖形語言:S2/S3 交叉相撞 → S5 平行不相交

## Negative Prompt

- 不用紫藍漸層、gradient text、neon cyan、左邊框條、純黑純白
- 不用 pie chart、dashboard 六宮格、圖例與刻度
- 不用行銷口號式語氣;風險段落不可草率帶過
- 不可在非最終場景加出場動畫;裝飾不可靜止
