---
name: gmail-ai-digest
description: Use when the user asks to整理 Gmail 未讀重要信件、生成 Gmail AI Digest、按 finance/news/research/other 分類摘要、套用白名單來源、輸出 Markdown，並寫入 Notion 的 gmail信件重點摘要資料庫。
---

# Gmail AI Digest

整理 Gmail 未讀重要信件，按固定白名單與分類輸出新聞摘要型 digest，並寫入 Notion。

## Scope

- Default timezone: Asia/Taipei.
- Default window: today in Taipei. If Gmail `after:/before:` misses UTC-boundary messages, use `newer_than:1d`, then state `日期：YYYY-MM-DD（近 24 小時）`.
- Only include unread Gmail messages matching the whitelist below.
- Treat "important" as the user-defined whitelist unless the user explicitly requires Gmail `IMPORTANT`.
- Do not invent content. If a category has no matching emails, still create a digest saying no matching emails.

## Categories And Whitelist

Use only these category values in the digest content:

- `finance`
- `news`
- `research`
- `other`

Whitelist mapping:

- `finance`
  - 鉅亨
- `news`
  - 生策會
  - nytimes.com
  - newmx.cnyes.com
  - statementdog.com
  - emarketing.fbs.com.tw
- `research`
  - Springer
  - Medscape
  - Journal of Medical Systems
  - smart pharmacy
  - telepharmacy
- `other`
  - Medium

## Gmail Search Workflow

1. Search each category separately with Gmail `search_emails`.
2. Start with strict Taipei-date queries:
   - `is:unread after:YYYY/M/D before:YYYY/M/D <whitelist terms>`
3. If strict date queries return empty but the user expects a daily digest, run a fallback:
   - `is:unread newer_than:1d <whitelist terms>`
4. For domain whitelist entries, search both `from:domain` and the bare domain keyword if needed.
5. Use `batch_read_email` for shortlisted messages before summarizing.
6. De-duplicate by message id, subject, canonical article title, and repeated newsletter links.

Useful query groups:

```text
finance: is:unread newer_than:1d ("鉅亨" OR cnyes)
news: is:unread newer_than:1d (nytimes OR "New York Times" OR newmx.cnyes.com OR statementdog.com OR emarketing.fbs.com.tw OR "生策會")
research: is:unread newer_than:1d (Springer OR Medscape OR "Journal of Medical Systems" OR "smart pharmacy" OR telepharmacy)
other: is:unread newer_than:1d Medium
```

## Output Format

For each category, write one Markdown digest using exactly this structure:

```markdown
日期：YYYY-MM-DD
分類：finance|news|research|other

### 今日重點（5–8 條）

1. **重點標題：** 一句完整中文摘要。
2. **重點標題：** 一句完整中文摘要。

### 新聞清單（可點連結）

* **[原文標題](https://example.com/article)**
  50 字內中文摘要，說明文章主旨與重要性。
  **原始連結：** https://example.com/article

### 原始內容關鍵字

keyword1, keyword2, keyword3.
```

If there are fewer than 5 meaningful items, include only real items. If no items:

```markdown
日期：YYYY-MM-DD
分類：category

### 今日重點（5–8 條）

1. **今日無符合 category 白名單信件：** 以 Gmail 未讀信、指定日期/近 24 小時視窗、category 來源白名單查詢，未找到可整理的信件。

### 新聞清單（可點連結）

* **今日無 category 類未讀重要信件**
  本分類今日沒有符合來源白名單與未讀條件的內容，因此不產生摘要與原始連結。
  **原始連結：** 無

### 原始內容關鍵字

category, unread Gmail, no matching emails, YYYY-MM-DD.
```

## Notion Write

Write each digest into the Notion database `gmail信件重點摘要` when available.

Known data source:

- `collection://1d4e979d-cae8-801c-8c22-000bba89f270`

Use these database properties:

- `信件標題`: `Gmail AI Digest - YYYY-MM-DD - category`
- `信件分類`: map `finance` -> `金融`, `news` -> `新聞`, `research` -> `研究`, `other` -> `其他`
- `重要程度`: `中` when items exist, `低` when no items exist
- `後續處理狀態`: `待處理` when items exist, `不需處理` when no items exist
- `需採取行動`: `__YES__` when items exist, `__NO__` when no items exist
- `主要內容`: one concise Chinese summary of the digest
- `相關標籤`: choose from existing options such as `財務`, `工作`, `學習`, `健康`

If the database cannot be found, search Notion for `gmail信件重點摘要`, fetch it to confirm the data source schema, then create pages under the data source.

## Chunking

If a generated digest exceeds 2000 Chinese characters, split it into chunks by category and section:

- Part 1: 今日重點
- Part 2+: 新聞清單
- Final part: 原始內容關鍵字

Keep date and category at the top of every chunk.
