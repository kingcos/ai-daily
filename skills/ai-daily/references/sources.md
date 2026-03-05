# Information Sources

## Tier 1 — Official AI Company Blogs

Higher importance weight. Apply +0.5 bonus to `importance` when score ≥ 3.

| Source | URL |
|--------|-----|
| Anthropic | https://www.anthropic.com/news |
| OpenAI | https://openai.com/news |
| Google DeepMind | https://deepmind.google/discover/blog/ |
| Cursor | https://cursor.com/blog |

**Fetch method:** Parse HTML article list or RSS if available. Extract title, URL, published date.

## Tier 2 — Tech Media (English)

| Source | URL / Endpoint | Notes |
|--------|----------------|-------|
| TechCrunch (AI) | https://techcrunch.com/tag/artificial-intelligence/feed/ | RSS |
| The Verge (AI) | https://www.theverge.com/rss/ai-artificial-intelligence/index.xml | RSS |
| Hacker News | https://hacker-news.firebaseio.com/v0/topstories.json | Fetch top 100 stories; filter by AI keywords |
| MIT Technology Review | https://www.technologyreview.com/feed/ | RSS |

**Hacker News AI filter keywords:**
```
AI, LLM, GPT, Claude, Gemini, Anthropic, OpenAI, model, agent, RAG, fine-tun
```
Fetch item details at `https://hacker-news.firebaseio.com/v0/item/{id}.json`. Use item `url` as `source_url` (not HN link). Set `source_name: "Hacker News"`.

## Tier 3 — Chinese AI Media

Items from these sources use `lang: "zh"` and follow the ZH-only schema.

| Source | URL | Fetch Method |
|--------|-----|--------------|
| 量子位 | https://www.qbitai.com | Crawl article list page |
| 机器之心 | https://www.jiqizhixin.com | Crawl article list page |

**Crawl instructions:**
- Fetch the article list/homepage HTML
- Extract: title (`title_zh`), article URL (`source_url`), published time
- No stable RSS — parse HTML directly
- Suggested frequency: once per day
- Route these items through the ZH processing branch (skip `title_en`, `summary_en`)

## Source Name Values

Use these exact strings in `source_name`:

| Display | `source_name` value |
|---------|---------------------|
| Anthropic blog | `"Anthropic"` |
| OpenAI blog | `"OpenAI"` |
| Google DeepMind blog | `"Google DeepMind"` |
| Cursor blog | `"Cursor"` |
| TechCrunch | `"TechCrunch"` |
| The Verge | `"The Verge"` |
| Hacker News | `"Hacker News"` |
| MIT Technology Review | `"MIT Technology Review"` |
| 量子位 | `"量子位"` |
| 机器之心 | `"机器之心"` |

## Planned Sources (Not Yet Implemented)

| Source | Status | Notes |
|--------|--------|-------|
| X / Twitter | ⏳ TODO | 跟踪 AI 相关热门推文/话题；抓取方案待定 |
