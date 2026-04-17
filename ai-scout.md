# /ai-scout — AI 情报定时采集

## 执行目标
从多个信息源并行采集 AI/Agent 最新动态，输出到 Obsidian Vault 暂存区。

## 执行前准备
1. 读取 D:\BuildingAgent_forobsidian\BuildingAgent\CLAUDE.md 了解 Vault 规范
2. 读取 D:\BuildingAgent_forobsidian\BuildingAgent\03_Watchlist\ 下的所有文件，优先采集 Watchlist 中的对象
3. 读取 D:\BuildingAgent_forobsidian\BuildingAgent\weights.json 了解当前权重偏好

## 采集任务（按顺序执行）

### 任务1：GitHub Trending
使用 web_search 搜索以下查询：
- "github trending today AI agent"
- "github trending today MCP LLM"
- "github trending today prompt engineering"

对 Watchlist 中的每个 GitHub 仓库，额外搜索其最新 release 和 star 动态。

每条记录必须包含：
- repo 名称和 URL
- 今日新增 star 数（如能获取）
- 仓库描述
- 来源：GitHub

### 任务2：Hacker News
使用 web_fetch 获取：`https://hn.algolia.com/api/v1/search?query=agent+LLM&tags=story&hitsPerPage=20`

补充搜索：
- web_search: "hacker news agent MCP site:news.ycombinator.com"
- web_search: "hacker news Claude Code site:news.ycombinator.com"

每条记录必须包含：标题、points、评论数、URL、发布时间。来源：HN

### 任务3：Reddit
使用 web_search 分别搜索：
- "site:reddit.com/r/LocalLLaMA agent MCP after:today"
- "site:reddit.com/r/MachineLearning Claude agent after:today"
- "site:reddit.com/r/artificial building agent after:today"

每条记录包含：标题、评论数估算、URL。来源：Reddit

### 任务4：X / Twitter（尽力采集，质量不稳定属正常）
使用 web_search 搜索：
- "site:x.com OR site:twitter.com Claude Code agent 2026"
- "site:x.com OR site:twitter.com MCP server agent skill"
- "site:x.com OR site:twitter.com Anthropic OpenAI agent"

对 Watchlist/KOL.md 中列出的账号，优先搜索其近期内容。

每条记录包含：账号、内容摘要、URL（如能获取）。来源：X/Twitter

### 任务5：大厂官方（必须执行）
使用 web_fetch 或 web_search：
- Anthropic Blog: 搜索 "site:anthropic.com/news"
- OpenAI Blog: 搜索 "site:openai.com/news"
- 对 Watchlist/竞品.md 中列出的产品，搜索近期 AI 功能更新

每条记录包含：标题、来源平台、URL、发布时间。

## 输出格式
将所有采集结果写入：
D:\BuildingAgent_forobsidian\BuildingAgent\00_Inbox\raw_[YYYY-MM-DD].md

格式如下：
```
---
scout_date: YYYY-MM-DD HH:MM
total_raw: N
sources: [GitHub, HN, Reddit, X, 大厂官方]
---

# Raw Inbox - YYYY-MM-DD

## GitHub
- **[repo名](URL)** | ⭐今日新增: N | 描述: xxx

## Hacker News
- **[标题](URL)** | Points: N | 评论: N | 时间: 相对时间

## Reddit
- **[标题](URL)** | 评论: N | 来源: r/subreddit名

## X / Twitter
- **[@账号]**: 内容摘要 [URL]

## 大厂官方
- **[Anthropic/OpenAI/其他]**: [标题](URL) | 时间: YYYY-MM-DD
```

## 完成后
输出执行摘要：采集完成，共 N 条原始信息，已写入 00_Inbox/raw_[日期].md。
