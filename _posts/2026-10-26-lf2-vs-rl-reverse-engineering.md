---
title: "用 CDP 逆向 Little Fighter 2 Remastered：打造 RL 訓練環境的實錄（上）"
date: 2026-10-26 12:30:00 +0800
categories: [LF2]
tags: [逆向工程, CDP, 強化學習, Python]
---

[Little Fighter 2](https://en.wikipedia.org/wiki/Little_Fighter_2) 是很多人的童年。我最近的專案 [lf2-bot](https://github.com/HuangBo-Chien/lf2-bot) 是用 Python 3.12 寫的 bot，直接**驅動 Steam 上的 LF2 Remastered（Electron 版）來打 VS 模式**，最終目標是拿它當強化學習的訓練環境。

這篇整理幾個最有意思的發現。所有機制都有實機證據（probe → record → verify 流程，見文末）。

## 怎麼控制一個沒有 API 的遊戲？

LF2 Remastered 本體是一個 Electron 應用。給它加上 `--remote-debugging-port` 啟動，就能用 **Chrome DevTools Protocol** 連上 renderer：

- **讀狀態**：用 `Runtime.evaluate` 注入 JS，直接讀遊戲物件（角色座標、HP、frame_state……）
- **下指令**：用 `Input.dispatchKeyEvent` 注入鍵盤事件，P1 是 WASD + JKL

遊戲邏輯全部在一個混淆過的 `px.js` 裡，欄位名是 `H0`、`ts`、`Eh` 這種。要做的事就是把它們一個個對應回語意名稱。

## 發現 1：視窗被蓋住，整個遊戲就凍結

Electron 視窗一旦最小化或被蓋住，`document.hidden` 變 true → `requestAnimationFrame` 停止 → **遊戲主迴圈整個凍結**，輸入和輪詢全部卡死。這不是「變慢」，是完全停住。

解法是 OS 層級把視窗還原（Electron 版連 `Page.bringToFront` 都沒用）。這個坑讓訓練跑一整夜回來發現只跑了十分鐘的人（我）記得很清楚。

## 發現 2：投擲的完整輸入表

「按 attack 會發生什麼」由三個變數完全決定：**手持物編碼 `Eh` × 移動狀態（站立/跑步/空中）× 方向鍵是否按住**。整理成表：

| 手持物 | 站著按 attack | 按住方向＋attack | 跑步中＋方向＋attack |
|---|---|---|---|
| 石頭 / 木箱 | 丟 | 丟 | 丟 |
| 棒球 / 空瓶 | 丟 | 丟 | 丟 |
| 木棍 / 鋤頭 | 揮擊 | 揮擊 | **丟（必須先起跑）** |
| 小刀 | 揮擊 | **丟** | 丟 |
| 奶 / 酒 | **喝** | 喝 | 丟 |

有趣的細節：

- 奶「站著按＝喝、跑步＋方向＝丟」——所以 bot 要喝奶前必須**先放開所有方向鍵**，不然會把奶丟出去。
- 木棍/鋤頭丟擲需要先進入跑步狀態（fs2），而跑步需要「真雙擊」方向鍵。

### 雙擊跑步的時序量測

機器人 double-tap 方向鍵，第一次嘗試是 0ms 間隔直接 tap 接 hold——**42,054 個輪詢裡一次都沒進入跑步狀態**。遊戲把極短間隔判成「按住」，是走路不是跑步。

回頭量測 78 個人類成功起跑的錄影：

- 第一按時長：p50 = **111ms**
- 放開到第二按：p50 = **99ms**，觀測最小 **47ms**

於是 bot 的配方定為：

```python
RUN_TAP_S = 0.10  # 第一按 100ms（人類 p50 中心）
RUN_GAP_S = 0.12  # 間隔 120ms
# release_all → press → 0.10s → release → beat → press+hold → 輪詢 fs==2
```

實機驗證：首次成功觀測到 fs2，打破 0/42,054。

## 發現 3：把東西丟出場外＝讓它消失的唯一方法

道具清理（清場）靠的是遊戲的邊界規則，每一幀檢查：

- **x 方向**：物品落地且 `x < 0` 或 `x > 場地寬度` → 直接從遊戲中刪除，不回收
- **z 方向**：永遠只是夾回邊界內，不刪除
- **被持有中**的物品跳過邊界檢查——所以可以站在牆邊，讓持有物懸在界外，離手瞬間才判定

於是「把場上的石頭清掉」的正確策略是：撿起來 → 走到場邊 → 朝外丟。而且投擲方向由**離手瞬間的面向**決定，而面向跟隨最後的行走方向——不用額外轉身操作，從哪個方向走進場邊就決定了丟向。

陷阱：道具雨的生成閘是「場上物品 < 4」，清掉一個馬上補一個——**紅皇后競賽**。真清場得先把道具雨關掉。

## 發現 4：frame_state 15 不是暈眩

早期把 frame_state 10–16 全當成 hitstun（被打的不可行動狀態），後來勘誤：**15 是「武器互動家族」**（撿取、投擲、持重物行走），不是被打。這個 bug 的後果是 bot 在自己投擲的動畫期間誤判「不可操作」而放棄輸入。已修正並有測試斷言守著。

## 發現 5：Steam 更新會輪替混淆名

2026-10-20 Steam 自動更新後，`px.js` 的混淆欄位名全部輪替（`H0→Q0`、`ts→ys`……），所有輪詢一夜之間讀到 None。現在核心欄位名集中在 `game_bindings.py` 管理，帶 BuildID 和證據標註；表達式的**輸出鍵**保持穩定，這樣消費端不用改。

教訓：綁定混淆名時，**集中、標註證據、隔離變動**。另外一旦輪詢全面讀 None，先懷疑又輪替了，probe 之後再修，不要用猜的。

## 方法論：probe → record → verify

所有遊戲機制的結論都走同一套流程：

1. **probe**：先小規模實驗，確認欄位可讀、行為可觀測
2. **record**：人類操作實錄（JSONL，每幀狀態＋按鍵）
3. **verify**：bot 重現操作，機器判讀是否成功

判決只有幾種：`ANSWERED`（證成）、`FALSIFIED`（否證）、`UNKNOWN` / `INCONCLUSIVE`（誠實的不知道）。A/B 實驗還會**預先登錄判準**再跑——有一次量測「斜跑跳踢」的攻擊距離帶，資料噪到判不出來，預登判準給出的判決是 WHIFFED（揮空），最後歸因發現是進場姿態問題，而不是攻擊帶太窄。沒有預登，就會忍不住事後找理由把結果解釋成「成功」。

## 然後呢

這套環境已經支撐了 Gymnasium 1v1 VS 環境、numpy 手寫的 Double-DQN、self-play 和難度課程訓練——包含一次誠實失敗的 20 小時課程梯（全梯未過門檻）。RL 這條線的細節，留給（下）篇。

原始文檔（更硬核、含 px.js 行號級證據）在 repo 的 `docs/`：[rl-env-findings.md](https://github.com/HuangBo-Chien/lf2-bot/blob/dev/docs/rl-env-findings.md)。
