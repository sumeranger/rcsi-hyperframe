# Design — RCSI 說服短片「鎖與光」

```yaml
name: Lock & Light
audience: 工程主管 / 技術決策者
colors:
  bg: "#0B0E14" # 深夜機房藍黑(冷色調,非純黑)
  bg-panel: "#121826" # 卡片/面板底
  fg: "#E8ECF4" # 主文字(冷白)
  fg-dim: "#8B94A7" # 次要文字/標籤
  alarm: "#E5484D" # 死結 / 阻塞 / 危機(血紅)
  alarm-deep: "#7F1D24" # 紅色深層光暈
  go: "#3DD68C" # RCSI / 通行 / 解方(號誌綠)
  go-deep: "#14532D" # 綠色深層光暈
  caution: "#F0B429" # 風險 / 注意(琥珀)
  steel: "#4A5878" # 結構線 / 圖表中性色
typography:
  display: # 戲劇性陳述(中文標題)
    fontFamily: '"Noto Serif CJK TC", "Noto Serif TC", serif'
    fontWeight: 900
    fontSize: 72-130px
  body: # 說明文字
    fontFamily: '"Noto Sans CJK TC", "Noto Sans TC", sans-serif'
    fontWeight: 350
    fontSize: 28-42px
  code: # SQL / 錯誤訊息 / 數據
    fontFamily: '"JetBrains Mono", "Noto Sans Mono CJK TC", monospace'
    fontWeight: 400-700
    fontSize: 22-36px
rounded:
  sm: 4px
  md: 10px
spacing:
  framePadding: 100-140px
  gap: 24-48px
motion:
  energy: medium-high(危機場景 high、機制講解 medium、結尾 calm)
  easing:
    entry: expo.out / power3.out / back.out(1.4)
    exit: power2.in(僅最終場景)
    ambient: sine.inOut
  transition: CSS 為主 — 主轉場 push slide / blur-through;危機→解方用 hard cut + 色彩反轉
```

## Overview

深夜資料庫機房的電影感。前半段(危機)由警報紅主導:deadlock graph、犧牲者被砍。
後半段(解方)切換為號誌綠:版本快照讓讀寫互不阻塞。風險段落用琥珀色誠實揭露。
觀眾是工程主管——畫面要有數據與證據感(monospace 指標、官方文件引用),不是行銷浮誇風。

## Colors

語意鎖定,不得混用:紅只給「鎖/阻塞/死結」,綠只給「RCSI/版本/通行」,琥珀只給「風險/代價」。
中性結構線用 steel,背景永遠 #0B0E14。

## Typography

- 中文標題用 Noto Serif CJK TC 900 — 襯線的莊重 = 生產環境事故的嚴肅性。
- 中文內文 Noto Sans CJK TC 350(深底加亮視覺增重,故用 350 非 400)。
- SQL、錯誤訊息、指標一律 monospace — 機器的聲音。serif(人的判斷)vs mono(機器的事實)是本片的字型張力。
- 中文不使用 letter-spacing 負值;行高 1.55+。數字用 tabular-nums。

## 張力說明

serif 講「故事與判斷」(凌晨三點、犧牲者、該不該開),mono 講「機器的事實」(error 1205、wait chain、version store bytes)。兩種聲音交錯 = 本片的敘事結構。

## Do's and Don'ts

- DO:每場景 8-10 個視覺元素;背景有 radial glow / 鎖形 ghost 圖示 / 網格;裝飾元素全部帶 ambient motion。
- DO:deadlock 視覺化要像「監控室即時畫面」,有 timestamp、spid、wait type 等 monospace 細節。
- DON'T:紫藍漸層、gradient text、左邊框 accent 條、純 #000/#fff、cyan neon。
- DON'T:行銷語氣。主管要的是誠實的工程判斷——風險段落必須與效益段落同等認真。
