# JSON Schema Reference

## Daily File: `data/YYYY-MM-DD.json` / `docs/data/YYYY-MM-DD.json`

```json
{
  "date": "2026-03-05",
  "generated_at": "2026-03-06T12:00:00Z",
  "items": [ /* array of item objects */ ]
}
```

---

## EN Item Schema (`lang: "en"`)

```json
{
  "id": "a3f9c21b",
  "lang": "en",
  "title_en": "Claude 4 Announced with Extended Thinking",
  "title_zh": "Claude 4 发布，支持扩展思考模式",
  "summary_en": "Anthropic: Anthropic announced Claude 4, featuring extended thinking capabilities and improved coding performance. The model sets new benchmarks on SWE-bench and introduces a new pricing tier. Now available via API and Claude.ai. (Official)",
  "summary_zh": "Anthropic：Anthropic 发布 Claude 4，支持扩展思考模式，编程能力大幅提升。新模型在 SWE-bench 上创下新纪录，并引入新定价方案。现已通过 API 和 Claude.ai 开放使用。（官方）",
  "tags": ["模型发布", "Anthropic", "推理"],
  "source_name": "Anthropic",
  "source_url": "https://www.anthropic.com/news/claude-4",
  "published_at": "2026-03-05T18:00:00Z",
  "collected_at": "2026-03-06T08:00:00Z",
  "importance": 5,
  "merged_sources": []
}
```

### EN Item Field Reference

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | First 8 hex chars of SHA-256 of canonical URL |
| `lang` | `"en"` | Source language |
| `title_en` | string | Original title, preserved verbatim |
| `title_zh` | string | AI-translated Chinese title (≤20 chars preferred) |
| `summary_en` | string | English summary in prescribed format (≤200 words) |
| `summary_zh` | string | Chinese summary in prescribed format (≤200 字) |
| `tags` | string[1-3] | 1–3 tags from taxonomy |
| `source_name` | string | Display name of primary source |
| `source_url` | string | Canonical URL of article |
| `published_at` | ISO8601 \| null | Verified original publish time (UTC); null if unverifiable |
| `collected_at` | ISO8601 | Pipeline run time (UTC) |
| `importance` | int 1–5 | AI-assigned score |
| `merged_sources` | string[] | URLs of duplicate articles merged into this item |

---

## ZH-Only Item Schema (`lang: "zh"`)

Used for items from 量子位 and 机器之心. Fields `title_en` and `summary_en` are **omitted**.

```json
{
  "id": "b7e2a10f",
  "lang": "zh",
  "title_zh": "OpenAI 发布 o3 模型，推理能力大幅提升",
  "summary_zh": "OpenAI：OpenAI 正式发布 o3 模型，在数学和编程基准测试中超越此前所有模型。该模型采用全新推理架构，支持更长的思维链。（量子位）",
  "tags": ["模型发布", "OpenAI", "推理"],
  "source_name": "量子位",
  "source_url": "https://www.qbitai.com/2026/03/xxxxx.html",
  "published_at": "2026-03-05T10:00:00Z",
  "collected_at": "2026-03-06T08:00:00Z",
  "importance": 4,
  "merged_sources": []
}
```

### ZH-Only Field Reference

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | First 8 hex chars of SHA-256 of canonical URL |
| `lang` | `"zh"` | Source language marker |
| `title_zh` | string | Original Chinese title, verbatim |
| `title_en` | — | **Omitted** |
| `summary_zh` | string | Chinese summary in prescribed format (≤200 字) |
| `summary_en` | — | **Omitted** |
| `tags` | string[1-3] | Same taxonomy as EN items |
| `source_name` | string | `"量子位"` or `"机器之心"` |
| `source_url` | string | Original article URL |
| `published_at` | ISO8601 \| null | Original publish time (UTC) |
| `collected_at` | ISO8601 | Crawl time (UTC) |
| `importance` | int 1–5 | Score using same rubric as EN items |
| `merged_sources` | string[] | Same-source merged URLs |

---

## Index File: `data/index.json` / `docs/data/index.json`

```json
{
  "latest_date": "2026-03-05",
  "updated_at": "2026-03-06T12:00:00Z",
  "dates": ["2026-03-05", "2026-03-04", "2026-03-03"],
  "items": [ /* same as latest daily file items */ ]
}
```

| Field | Notes |
|-------|-------|
| `latest_date` | Date string of the most recently generated daily file |
| `updated_at` | ISO8601 timestamp of last pipeline run |
| `dates` | Array of all dates with existing daily files, descending |
| `items` | Full items array from the latest daily file |

**Empty template** (before first run):
```json
{
  "latest_date": null,
  "updated_at": null,
  "dates": [],
  "items": []
}
```

---

## Cross-Language Deduplication Rule

EN and ZH articles may cover the same event. When they do:
- **Keep both as independent items** — do not merge
- `lang` differs (`"en"` vs `"zh"`), so they are treated as distinct entries
- The frontend may optionally group them for display
- Only merge within the same language (EN+EN or ZH+ZH)

---

## ID Generation

```
id = sha256(canonical_url)[0:8]
```

Where `canonical_url` is the full article URL, lowercased, with trailing slash removed.
