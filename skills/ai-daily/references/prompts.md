# AI Prompt Templates

All prompts are used during pipeline execution. Substitute `{placeholders}` with actual values.

---

## Deduplication Prompt

Used in Step 4 to determine if two articles cover the same event.

```
Are these two articles reporting on the same specific event or announcement?
Answer YES or NO only.

Article A: {title_a} — {url_a}
Article B: {title_b} — {url_b}
```

---

## ZH Summary Prompt

Used in Step 5 for `lang: "zh"` items (量子位, 机器之心).

```
根据以下新闻内容，生成一段中文摘要。

格式要求：
- 开头："{发布方机构名}："（注意：是发布事件的公司/机构，非媒体名）
- 正文：核心事实密集排列，优先包含原文中的具体数字和指标
- **结尾不加任何括号来源**，`source_name` 字段已单独记录来源
- 严格不超过200字（含标点），不分段

严格校验：
- 只使用原文中明确出现的事实和数据，不推断不估算
- 禁用：重磅、革命性、突破性、值得关注、首次（除非原文明确写明）
- 发布时间须与原文一致，来源须与实际报道方一致

标题：{title_zh}
来源：{source_name}
原文发布时间：{published_at}
正文：{article_text}
```

---

## EN Summary Prompt (for `summary_en`)

Used in Step 5 for `lang: "en"` items.

```
Write a news summary following this format exactly:

Format:
- Start: "{Publishing entity}:" (the company/org that made the announcement, NOT the media outlet)
- Body: dense factual content with specific numbers and metrics from the article
- **No trailing source bracket** — do NOT append "(Official)", "(The Verge)", etc. at the end; source is already in `source_name` field
- Max 200 words, no paragraph breaks

Strict rules:
- Only use data explicitly stated in the article — never infer or estimate
- No subjective adjectives: "groundbreaking", "revolutionary", "remarkable", etc.
- Dates must match the article's actual publish date
- Source must match the actual reporting outlet

Title: {title_en}
Source: {source_name}
Published: {published_at}
Content: {article_text}
```

---

## EN → ZH Summary Prompt (for `summary_zh` from EN source)

Used in Step 5 for `lang: "en"` items, to generate the Chinese summary.

```
根据以下英文新闻，生成一段中文摘要。

要求：格式为"{主体}：...。（{来源}）"，严格不超过200字，不分段，
只用原文明确出现的事实，禁止添加原文没有的内容，
禁止"重磅""革命性"等修饰词。

Title: {title_en}
Source: {source_name}
Published: {published_at}
Content: {article_text}
```

---

## Importance Scoring Prompt (EN)

Used in Step 7 for `lang: "en"` items.

```
Score this AI news item from 1-5 based on its significance to the AI industry.
5=major milestone (flagship model launch, industry-defining), 1=minor/peripheral.
Reply with a single integer only.

Title: {title_en}
Summary: {summary_en}
Source: {source_name}
```

---

## Importance Scoring Prompt (ZH)

Used in Step 7 for `lang: "zh"` items.

```
根据这条 AI 新闻对行业的重要程度，给出 1-5 分。
5=重大里程碑（旗舰模型发布、行业定义性事件），1=边缘/低价值。
只回复一个数字。

标题：{title_zh}
摘要：{summary_zh}
来源：{source_name}
```
