---
title: "開張：一個記錄 vibe coding 的部落格"
date: 2026-10-26 12:00:00 +0800
categories: [公告]
tags: [隨筆]
---

這個部落格開張了。

## 為什麼要開

從開始 vibe coding 以來，做了不少雜七雜八的東西：遊戲 bot、逆向工程、強化學習實驗、一次性的小工具……每個專案都累積了一大堆發現——有些是查了好幾天才確認的遊戲機制、有些是浪費了一整個下午才發現的死胡同。這些東西散落在各專案的 `docs/`、commit message 和對話記錄裡，時間一久連自己都找不到。

所以開這個部落格，把這些記錄整理成文章丟上來。**不只記成功，也要記失敗**——很多時候「這條路為什麼走不通」比「最後怎麼做成的」更有價值。

## 會寫什麼

- **專案筆記**：進行中工作的整理，例如 [lf2-bot](https://github.com/HuangBo-Chien/lf2-bot)（用 Python 透過 Chrome DevTools Protocol 玩 Little Fighter 2 的 bot）
- **逆向工程發現**：從混淆過的 JS 裡挖出來的遊戲機制
- **實驗記錄**：強化學習訓練、A/B 測試的結果，誠實包含負結果
- **失敗記錄**：被否證的假說、踩過的坑

## 這個部落格本身也是 vibe coding 出來的

Jekyll + Minimal Mistakes 主題、GitHub Pages 部署、giscus 留言——整個網站從無到有（含這篇文章）都是在 opencode 裡完成的。以後寫文章的流程也簡單：丟一個 Markdown 檔進 `_posts/`、push、等一分鐘上線。

順手展示一下這裡支援的格式。程式碼高亮：

```python
def vibe(coding: bool = True) -> str:
    """把腦中的想法變成能動的東西。"""
    return "it works!" if coding else "keep trying"
```

表格：

| 格式 | 語法 |
|---|---|
| 程式碼 | ` ```python ` 區塊 |
| 圖片 | `![說明](/assets/images/xxx.png)` |
| 引用 | `> 像這樣` |

> 寫部落格最好的時機是十年前，其次是現在。

下一篇文章會是 lf2-bot 逆向工程的實錄。敬請期待。
