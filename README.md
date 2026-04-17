# Claude AI Intel Skills

> 基于 Claude Code 的 AI/Agent 领域情报自动化工作流

专为 AI 产品经理设计。4 条自定义命令完成从**情报采集 → 过滤评分 → 生成简报 → 深度研究**的完整闭环，输出结果沉淀至 Obsidian Vault，并可选同步飞书多维表格。

---

## 目录

- [系统概览](#系统概览)
- [工作流全景](#工作流全景)
- [命令参考](#命令参考)
- [前置要求](#前置要求)
- [安装方法](#安装方法)
- [Vault 目录结构](#vault-目录结构)
- [配置文件说明](#配置文件说明)
- [使用示例](#使用示例)

---

## 系统概览

本项目是一套运行在 **Claude Code** 上的自定义命令（Custom Commands），面向需要持续追踪 AI/Agent 领域动态的产品经理和研究者。

**核心能力：**

| 能力 | 说明 |
|------|------|
| 多源情报采集 | 同时覆盖 GitHub、Hacker News、Reddit、X/Twitter、大厂官方博客 |
| 智能过滤评分 | 基于热度 × 时效双维度打分，自动去重、按时效分层 |
| 自动生成简报 | 输出结构化 Daily Digest，写入 Obsidian Vault |
| 飞书表格同步 | 可选配置，将每条情报条目同步至飞书多维表格 |
| 深度主题研究 | 按需触发，对特定主题进行多轮搜索与综合分析 |

---

## 工作流全景

```
/ai-scout  ──▶  /ai-filter-rank  ──▶  /ai-classify-digest  ──▶  /ai-research
    │                  │                       │                        │
    ▼                  ▼                       ▼                        ▼
采集原始情报       过滤 & 评分排序       生成简报 & 更新 Wiki        深度研究特定主题
    │                  │                       │
    ▼                  ▼                       ▼
00_Inbox/          00_Inbox/            01_Daily_Digest/
raw_*.md           filtered_*.md        YYYY-MM-DD.md
                                        02_Wiki/（追加更新）
                                        飞书多维表格（可选）
```

**标准执行顺序：**

```
步骤 1 → /ai-scout              采集原始情报
步骤 2 → /ai-filter-rank        过滤评分
步骤 3 → /ai-classify-digest    生成简报 & 同步飞书
步骤 4 → /ai-research [主题]    对感兴趣的主题深入研究（按需触发）
```

---

## 命令参考

| 命令 | 用途 | 读取 | 输出 |
|------|------|------|------|
| `/ai-scout` | 从 5 大平台采集最新 AI/Agent 动态 | Watchlist、weights.json | `00_Inbox/raw_*.md` |
| `/ai-filter-rank` | 过滤噪音、评分排序、时效分层 | `00_Inbox/raw_*.md` | `00_Inbox/filtered_*.md` |
| `/ai-classify-digest` | 生成简报、更新 Wiki、同步飞书 | `00_Inbox/filtered_*.md` | `01_Daily_Digest/*.md`、`02_Wiki/` |
| `/ai-research [主题]` | 多轮搜索 + 生成结构化研究报告 | 主题关键词 | `00_Inbox/research_*.md`、`02_Wiki/` |

---

### `/ai-scout` — 情报采集

从以下 5 个来源并行采集，优先覆盖 `03_Watchlist/` 中配置的追踪对象：

| 来源 | 采集内容 |
|------|---------|
| GitHub Trending | AI/Agent/MCP 相关仓库，含今日新增 star 数 |
| Hacker News | agent、LLM、Claude Code 相关热帖（points、评论数） |
| Reddit | r/LocalLLaMA、r/MachineLearning 等社区讨论 |
| X / Twitter | KOL 动态、Anthropic/OpenAI 官方账号近期内容 |
| 大厂官方 | Anthropic Blog、OpenAI Blog、Watchlist 竞品动态 |

**输出：** `00_Inbox/raw_YYYY-MM-DD.md`

---

### `/ai-filter-rank` — 过滤评分

读取最新 `raw_*.md`，执行三步处理：

**① 保留条件**（满足任意一项才保留）

- 技术创新：涉及新架构或新能力（multi-agent、MCP、memory、tool use 等）
- 产品启发：新交互方式、新工作流设计、新系统结构
- 趋势信号：多源重复出现 / 大厂正式发布 / GitHub 今日新增 star > 100
- 可复用方法：可提炼为 prompt 技巧 / agent 模式 / workflow 结构

**② 评分公式**

```
score = (hotness × 0.6) + (freshness × 0.4)
```

再乘以 `weights.json` 中对应分类的权重系数。

**③ 时效分层**

| 层级 | 发布时间 | 可进 Top 3？ | 最低保留分 |
|------|----------|-------------|-----------|
| 🔴 鲜活 | < 14 天 | ✅ 可以 | 无限制 |
| 🟡 近期参考 | 14 ~ 90 天 | ❌ | hotness ≥ 3.5 |
| 🟢 常青精选 | > 90 天 | ❌ | hotness ≥ 4.0 |

**分类标签（每条必须打一个）：**
`Agent Architecture` / `Prompt Engineering` / `Product Design` / `AI Infrastructure` / `Business Case`

**输出：** `00_Inbox/filtered_YYYY-MM-DD.md`

---

### `/ai-classify-digest` — 生成简报

依次执行以下步骤：

1. **确定 Top 3**：从「🔴鲜活」层中取 score 最高的 3 条，附产品视角分析（对 Building Agent 的可复用价值）
2. **生成 Daily Digest**：按模板输出至 `01_Daily_Digest/YYYY-MM-DD.md`，包含大厂动态、GitHub 趋势、趋势总结、建议跟进方向
3. **更新 Wiki**：score ≥ 4.0 的条目自动追加至 `02_Wiki/` 对应笔记（仅追加，不覆盖已有内容）
4. **同步飞书**：若已配置 `feishu_config.json`，将所有保留条目写入飞书多维表格（最多 500 条/次）
5. **清理 Inbox**：将 `raw_*.md` 和 `filtered_*.md` 归档至 `04_Archive/YYYY-MM/`

**输出：** `01_Daily_Digest/YYYY-MM-DD.md`、`02_Wiki/` 更新、飞书同步（可选）

---

### `/ai-research [主题]` — 深度研究

```bash
# 使用示例
/ai-research "MCP server 最新实现方案"
/ai-research "building agent 竞品分析"
/ai-research "multi-agent 协作架构最佳实践"
/ai-research "飞书多维表格 AI 集成 2026"
```

**执行步骤：**

1. 将主题拆解为 3-5 个具体搜索角度
2. 每个角度执行 web_search，对 2-3 个核心结果 web_fetch 深度阅读
3. 综合生成结构化研究报告：核心发现 / 对 Building Agent 的产品价值 / 推荐行动 / 信息来源
4. 报告写入 `00_Inbox/research_[主题关键词]_YYYY-MM-DD.md`，关键发现追加至对应 Wiki 笔记

**输出：** `00_Inbox/research_*.md`、`02_Wiki/` 追加更新

---

## 前置要求

| 依赖 | 是否必须 | 说明 |
|------|---------|------|
| Claude Code | ✅ 必须 | 命令运行环境，需安装并登录 |
| Obsidian | ✅ 必须 | 知识库管理，需按规定目录结构初始化 |
| `CLAUDE.md` | ✅ 必须 | 放于 Vault 根目录，定义路径和写入规范 |
| `weights.json` | ✅ 必须 | 放于 Vault 根目录，控制各分类评分权重 |
| `feishu_config.json` | ⬜ 可选 | 放于 Vault 根目录，配置飞书多维表格同步 |

---

## 安装方法

**第一步：复制命令文件**

将本仓库中的 4 个 `.md` 文件复制到 Claude Code 自定义命令目录：

```bash
# Windows
C:\Users\<用户名>\.claude\commands\ai-intel\

# macOS / Linux
~/.claude/commands/ai-intel/
```

**第二步：初始化 Obsidian Vault**

按以下结构创建目录：

```
YourVault/
├── CLAUDE.md                    ← 必须，Vault 路径和写入规范
├── weights.json                 ← 必须，评分权重（初始值见下方）
├── feishu_config.json           ← 可选，飞书同步配置
├── 00_Inbox/                    ← 原始信息暂存，Scout 输出到此
├── 01_Daily_Digest/             ← 每日简报输出目录
├── 02_Wiki/                     ← 主题知识库，自动追加更新
├── 03_Watchlist/                ← 重点追踪对象（KOL.md、竞品.md 等）
├── 04_Archive/                  ← 已处理文件自动归档
└── 99_Templates/                ← 模板文件，不可修改
    ├── daily_digest_template.md
    └── wiki_note_template.md
```

**第三步：创建配置文件**

创建 `weights.json`（复制以下内容）：

```json
{
  "Agent Architecture": 1.0,
  "Prompt Engineering": 1.0,
  "Product Design": 1.0,
  "AI Infrastructure": 1.0,
  "Business Case": 1.0
}
```

**第四步：在 Claude Code 中运行**

```
/ai-scout
```

---

## 配置文件说明

### `CLAUDE.md`（必须）

告知 Claude 以下信息：Vault 的绝对路径、各目录用途、写入规则（ISO 日期格式、WikiLinks 格式、来源必须注明 URL）、核心关注关键词。

放置位置：Vault 根目录。

---

### `weights.json`（必须）

控制各分类条目的评分权重，影响 `score` 最终值。

```json
{
  "Agent Architecture": 1.0,
  "Prompt Engineering": 1.0,
  "Product Design": 1.0,
  "AI Infrastructure": 1.0,
  "Business Case": 1.0
}
```

| 参数 | 范围 | 说明 |
|------|------|------|
| 各分类权重 | 0.5 ~ 2.0 | 权重越高，该类别条目在排序中越靠前 |

可通过 `/ai-update-weights [类别] [👍/👎/⭐]` 命令动态调整（👍/⭐ → +0.1，👎 → -0.1）。

---

### `feishu_config.json`（可选）

配置后，`/ai-classify-digest` 会将所有保留条目同步至飞书多维表格。

```json
{
  "app_id": "cli_xxxx",
  "app_secret": "xxxx",
  "bitable_app_token": "你的多维表格 app_token",
  "bitable_table_id": "你的 table_id",
  "bitable_enabled": true,
  "webhook_url": "",
  "webhook_enabled": false
}
```

| 字段 | 说明 |
|------|------|
| `app_id` / `app_secret` | 飞书自建应用的凭证，在飞书开放平台创建 |
| `bitable_app_token` | 多维表格 URL 中的唯一标识 |
| `bitable_table_id` | 目标数据表 ID |
| `bitable_enabled` | 设为 `false` 可随时禁用同步，不影响其他功能 |

飞书多维表格字段会在首次运行时自动创建，无需手动配置列结构。

---

## 使用示例

**标准每日工作流（约 5 分钟完成）：**

```
/ai-scout
→ 采集完成，共 47 条原始信息，已写入 00_Inbox/raw_2026-04-17.md

/ai-filter-rank
→ 过滤完成，原始 47 条 → 保留 18 条（丢弃 29 条），已写入 filtered_2026-04-17.md

/ai-classify-digest
→ 简报已生成 → 01_Daily_Digest/2026-04-17.md | Wiki 更新 3 条 | 飞书推送：成功

/ai-research "Claude Code MCP 最新进展"   ← 按需触发，不是每次都跑
→ 研究报告写入 00_Inbox/research_Claude-Code-MCP_2026-04-17.md
```

---

## 关键词覆盖范围

```
agent · skill · MCP · Claude Code · LangChain · multi-agent · CrewAI
building agent · prompt engineering · AGENTS.md · agentic
Anthropic · OpenAI · Notion AI · 飞书 · workflow automation
subagent · tool use · memory · RAG · fine-tuning
```

---

## License

MIT
