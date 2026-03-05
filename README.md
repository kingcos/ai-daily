# AI 日报

每日自动采集、去重、摘要并归档 AI 行业资讯，驱动一个静态 GitHub Pages 网站。

## 功能

- **新闻采集**：Anthropic、OpenAI、Google DeepMind、Cursor、TechCrunch、The Verge、Hacker News、MIT Technology Review、量子位、机器之心
- **智能去重**：语义去重，合并同一事件的不同来源报道
- **双语摘要**：自动生成中英文摘要（≤200字/词，严格事实导向）
- **结构化归档**：每日一个 JSON 文件，持续追加
- **静态前端**：支持深色模式、标签/来源/语言过滤、移动端适配

## 使用方式

在项目目录中打开 Claude Code，说：

> "帮我运行今天的 AI 日报流水线"

或

> "生成 2026-03-05 的日报数据"

Claude 会自动加载 `ai-daily` skill，按照 Fetch → Filter → Dedup → Verify → Summarize → Tag → Score → Write 的顺序执行，将结果写入 `data/YYYY-MM-DD.json` 并更新 `data/index.json`。

## 目录结构

```
ai-daily/
├── .claude-plugin/
│   └── plugin.json                 # Claude Code 插件元数据
├── skills/
│   └── ai-daily/
│       ├── SKILL.md                # Skill 定义（触发条件 + 管道概览）
│       └── references/
│           ├── sources.md          # 信息源清单
│           ├── schema.md           # JSON Schema
│           ├── pipeline.md         # 管道步骤详细说明
│           └── prompts.md          # AI 提示词模板
├── data/
│   ├── index.json                  # 最新一期数据（前端读取入口）
│   └── YYYY-MM-DD.json             # 每日归档文件
├── docs/
│   └── index.html                  # GitHub Pages 静态前端
└── README.md
```

## 数据格式

### `data/index.json`

```json
{
  "latest_date": "2026-03-05",
  "updated_at": "2026-03-06T12:00:00Z",
  "dates": ["2026-03-05", "2026-03-04"],
  "items": [ /* 最新一天的所有条目 */ ]
}
```

### 每条新闻条目（EN）

```json
{
  "id": "a3f9c21b",
  "lang": "en",
  "title_en": "Claude 4 Announced",
  "title_zh": "Claude 4 发布",
  "summary_zh": "Anthropic：...",
  "summary_en": "Anthropic: ...",
  "tags": ["模型发布", "Anthropic"],
  "source_name": "Anthropic",
  "source_url": "https://...",
  "published_at": "2026-03-05T18:00:00Z",
  "collected_at": "2026-03-06T08:00:00Z",
  "importance": 5,
  "merged_sources": []
}
```

中文源（量子位/机器之心）条目省略 `title_en` 和 `summary_en`，`lang` 为 `"zh"`。

## GitHub Pages 部署

1. 推送代码到 GitHub
2. 仓库 Settings → Pages → Source 选择 **`docs/` 目录**
3. 访问 `https://<username>.github.io/ai-daily/`

本地预览：在仓库根目录运行 `python3 -m http.server 8000`，访问 `http://localhost:8000/docs/`
