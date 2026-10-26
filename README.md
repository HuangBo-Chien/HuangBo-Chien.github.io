Huang Bo-Chien 的部落格 — vibe coding 記錄
=============================================

Jekyll + [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)（`remote_theme`），部署在 GitHub Pages。
**不需要本地裝 Ruby 或任何建置工具**：push 到 `main` 之後 GitHub 會自動 build，約 1–2 分鐘上線。

網址：<https://huangbo-chien.github.io/>

## 怎麼加一篇新文章

1. 複製下面的 front matter 範本，存成 `_posts/YYYY-MM-DD-english-slug.md`
   （檔名**日期前綴是必須的**，slug 建議用英文，會變成網址的一部分）
2. 下面直接寫 Markdown，push 到 `main`，等 1–2 分鐘就上線。

```markdown
---
title: "文章標題"
date: 2026-10-26 12:00:00 +0800
categories: [LF2]           # 大分類，一或多個
tags: [逆向工程, CDP]        # 小標籤，自由發揮
---

正文這裡開始。
```

### 注意事項

- **未來日期的文章不會出現**（Jekyll 預設行為）——date 不要填到未來。
- 分類/標籤頁（`/categories/`、`/tags/`）會自動收集 front matter 裡的值。
- 程式碼高亮：用 fenced code block 加語言名，如 ` ```python `。
- 圖片：丟到 `assets/images/`，文章裡用 `![說明文字](/assets/images/檔名.png)`。
- 留言（giscus）每篇文章自動帶入，不用另外設定。

## 想本地預覽的話（選配）

平常不需要。真的想本地看：裝 Ruby + Jekyll 後在 repo 根目錄跑 `bundle exec jekyll serve`，
或用 Docker：`docker run --rm -p 4000:4000 -v ${PWD}:/site bretfisher/jekyll-serve`。

## Build 失敗怎麼辦

GitHub Pages build 失敗時網站會停在舊版本。到 repo 的
**Settings → Pages → Build and deployment** 看錯誤訊息，
最常見的原因是 front matter 的 YAML 打錯（冒號後面要有空格、引號沒成對）。

## 網站設定

- 標題、作者、主題配色、留言：改 `_config.yml`（改完要重新 push 才生效）。
- 導覽列：改 `_data/navigation.yml`。
- 深色/淺色主題：`_config.yml` 的 `minimal_mistakes_skin`（"dark" / "default" / 其他）。
- 留言系統換 repo 時：更新 `repository`、`comments.giscus.repo_id`、`category_id`
  （ID 到 <https://giscus.app> 產生）。
