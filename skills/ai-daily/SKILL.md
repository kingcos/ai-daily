---
name: ai-daily
description: >
  Fetch, deduplicate, summarize, and archive AI industry news into a structured daily JSON file.
  Use this skill whenever the user wants to: collect or update AI news, run the daily or weekly
  news pipeline, process articles from AI company blogs or tech media, generate the daily JSON
  data file for the AI daily website, or when they mention "AI 日报", "news pipeline",
  "weekly digest", "update the feed", "抓取新闻", "生成日报", "更新数据", or "run the pipeline".
version: 1.0.0
---

# AI Daily News Pipeline

Collect, process, and archive AI industry news into structured daily JSON files that power a static GitHub Pages website.

## Pipeline Overview

Execute these stages **in order** for a given target date:

```
Fetch → Filter → Semantic Dedup → Time Verify → Summarize (EN+ZH) → Tag → Score → Write JSON
```

**Output files:**
- `data/YYYY-MM-DD.json` — one file per day
- `data/index.json` — always reflects the latest day's items + metadata

**Default target date:** today (UTC). Accept explicit dates from the user (e.g. "run for 2026-03-05").

## Quick Step Reference

| Step | Action | Detail |
|------|--------|--------|
| 1. Fetch | Collect articles from all sources | See `references/sources.md` |
| 2. Filter | Drop ads, job posts, off-topic | Discard if title/URL contains `sponsored`, `advertis`, `partner content`, `giveaway` |
| 3. Dedup | Merge same-event articles | Keep Tier 1 source; move other URL to `merged_sources[]` |
| 4. Time Verify | Confirm `published_at` | Check HTML meta → RSS pubDate → null if unknown |
| 5. Summarize | Generate ZH/EN summaries | See `references/prompts.md` for prompt templates |
| 6. Tag | Assign 1–3 tags | See `references/pipeline.md` Tag Taxonomy |
| 7. Score | Set `importance` 1–5 | Tier 1 source with score ≥3 gets +0.5 bonus (cap at 5) |
| 8. Write | Merge into daily JSON + rebuild index | Incremental by default; skip existing `id`s |

## Running the Pipeline

**Incremental run (default):**
1. Read existing `data/YYYY-MM-DD.json` if it exists
2. Fetch sources, process only new articles not yet in the file
3. Append new items, sort by `published_at` descending
4. Rebuild `data/index.json`

**Full re-run:**
- Delete the daily JSON first, then run the pipeline; all articles reprocessed

## Key Constraints

- Articles outside the target date window (00:00–23:59 UTC) are discarded after time verification
- EN and ZH articles covering the same event are **kept as separate items** (not merged)
- Never leave `tags: []` — use closest taxonomy match when uncertain
- Default `importance` to 3 when ambiguous
- `summary_en` / `title_en` are **omitted** for ZH-only items (`lang: "zh"`)

## Reference Files

Load these as needed during pipeline execution:

- `references/sources.md` — complete source list with URLs (Tier 1/2/3)
- `references/schema.md` — full JSON schema for EN items, ZH items, and index.json
- `references/pipeline.md` — detailed instructions for each pipeline step
- `references/prompts.md` — all AI prompt templates (dedup, summarize, score)
