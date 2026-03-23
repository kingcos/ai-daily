# Pipeline Step Instructions

## Step 1: Fetch

Collect articles from all configured sources (see `sources.md`).

**English sources (RSS/API):**
- Parse RSS feeds: extract `<pubDate>` or `<dc:date>` for publish time
- Hacker News: fetch `https://hacker-news.firebaseio.com/v0/topstories.json`, take top 100 IDs, then for each fetch `https://hacker-news.firebaseio.com/v0/item/{id}.json`. Filter titles/URLs by AI keywords: `AI, LLM, GPT, Claude, Gemini, Anthropic, OpenAI, model, agent, RAG, fine-tun`
- Set `lang: "en"` for all English sources

**Chinese sources (crawl):**
- 量子位：fetch `https://www.qbitai.com`, parse article list, extract title, URL, publish time
- 机器之心：fetch `https://www.jiqizhixin.com`, parse article list, extract title, URL, publish time
- Set `lang: "zh"` and route through ZH processing branch

**Collect for each item:** `title`, `url`, `published_at` (raw), `source_name`, `lang`

---

## Step 2: Filter

Discard items that match ANY of the following:

- Title or URL contains: `sponsored`, `advertis`, `partner content`, `giveaway`
- Content is purely a product listing, job posting, or opinion piece unrelated to AI technology
- Published date is outside the target date window (handled after Step 3)

---

## Step 3: Time Verification

Goal: Ensure `published_at` reflects the original publish time, not syndication/update time.

1. Parse raw date from feed metadata
2. If ambiguous or missing, fetch the article page and look for (in priority order):
   - `<meta property="article:published_time" content="...">`
   - `<time datetime="...">`
   - JSON-LD: `{"@type": "Article", "datePublished": "..."}`
3. If still unverifiable, set `"published_at": null` — do **not** discard the item
4. Discard articles with a verified `published_at` outside the target date window (00:00–23:59 UTC)

---

## Step 4: Semantic Deduplication

Goal: Merge articles from different sources that cover the same specific event.

**Pre-filter (no AI call needed):**
- Exact URL matches → auto-deduplicate immediately

**AI deduplication (for non-identical URLs):**
- Compare article pairs using the deduplication prompt (see `prompts.md`)
- Criteria: same core event AND published within 48 hours of each other
- If duplicate detected:
  - Keep the **Tier 1 source** as primary; if same tier, keep the **earliest** published
  - Move the other URL to `merged_sources[]`
  - Regenerate summary incorporating unique details from the merged source

**Cross-language rule:** Never merge EN and ZH items even if they cover the same event.

---

## Step 5: Summarize

### Summary Style Rules (applies to all summaries)

**Format template:**
```
{主体}：{核心事实，含具体数据}。{技术/产品亮点}。{落地/上线情况（如有）}。
```

**Rules:**
- Strict ≤200 字 (ZH) / ≤200 words (EN), no paragraph breaks
- Opening `{主体}：` — the organization making the announcement, NOT the media outlet
- **NO trailing source bracket** — do NOT append `（官方）`、`（The Verge）`、`（量子位）` or any source attribution at the end; source is already captured in `source_name` field
- Only state facts explicitly from the source article; never infer or estimate
- Forbidden adjectives: `重磅`、`革命性`、`突破性`、`值得关注`、`首次` (unless the original article uses them)
- Technical terms follow original article spelling: MoE, RAG, SWE-bench, etc.

### EN items (`lang: "en"`)

Generate three fields:
- `summary_zh` — Chinese summary following style rules (≤200 字)
- `summary_en` — English summary following style rules (≤200 words)
- `title_zh` — Chinese translation of the title (≤20 chars preferred)

Use prompts from `prompts.md`: `[ZH summary from EN]` and `[EN summary]`.

### ZH items (`lang: "zh"`)

Generate one field only:
- `summary_zh` — Chinese summary following style rules (≤200 字)
- **Skip** `summary_en` and `title_en` entirely

Use prompt from `prompts.md`: `[ZH summary]`.

---

## Step 6: Tag Extraction

Assign **1–3 tags** from the taxonomy below. Prefer specificity.

### Tag Taxonomy

| Category | Tags |
|----------|------|
| Event type | `模型发布`, `产品更新`, `研究论文`, `融资`, `收购`, `政策监管`, `开源` |
| Technology | `推理`, `多模态`, `Agent`, `RAG`, `微调`, `评测基准`, `安全对齐` |
| Company | `Anthropic`, `OpenAI`, `Google`, `Meta`, `Mistral`, `xAI`, `Cursor` |
| Domain | `编程`, `医疗`, `教育`, `法律`, `硬件` |

**Rules:**
- Max 3 tags per item
- Include at least 1 "Event type" tag when applicable
- Never use generic tags like `AI` or `机器学习`
- Never leave `tags: []` — use the closest match

---

## Step 7: Importance Scoring

Assign `importance` as integer 1–5:

| Score | Criteria |
|-------|----------|
| 5 | Major model/product launch from a top lab; industry-defining announcement |
| 4 | Significant update, notable research, large funding (>$100M) |
| 3 | Meaningful product update, interesting research, mid-size news |
| 2 | Minor update, niche tool, small announcement |
| 1 | Tangential, low-signal, mostly speculative |

**Tier 1 source bonus:** if `source_name` is a Tier 1 source AND raw score ≥ 3, add 0.5 and round up. Cap at 5.

**Default:** When uncertain, use `3`.

---

## Step 8: Write JSON

1. Load existing `data/YYYY-MM-DD.json` if it exists (for incremental runs)
2. Compute `id` for each new item: `sha256(source_url)[0:8]`
3. Skip any item whose `id` already exists in the file
4. Append new items
5. Sort all `items` by `published_at` descending (null values go to end)
6. Write the updated daily file to `data/YYYY-MM-DD.json`, then copy it to `docs/data/YYYY-MM-DD.json`
7. Rebuild `data/index.json`:
   - `latest_date` = target date
   - `updated_at` = current UTC time
   - `dates` = sorted list of all existing daily file dates (descending)
   - `items` = items from the latest daily file
8. Copy `data/index.json` to `docs/data/index.json` so GitHub Pages can serve it

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Article URL unreachable | Summarize from title only; prefix `summary_en` with `[title-only]` |
| `published_at` unverifiable | Set to `null`; do not discard the item |
| Deduplication uncertain | Keep both items; do not merge |
| Tag taxonomy miss | Use closest match; never leave `tags: []` |
| Importance ambiguous | Default to `3` |
| Source crawl fails | Log the failure; continue with remaining sources |
