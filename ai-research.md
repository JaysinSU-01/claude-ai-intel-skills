# /ai-research [主题] — 主题深度研究

## 使用方式
/ai-research "你想深度研究的主题"

示例：
- /ai-research "building agent 竞品分析"
- /ai-research "MCP server 最新实现方案"
- /ai-research "飞书多维表格 AI 集成 2026"
- /ai-research "multi-agent 协作架构最佳实践"

## 执行流程

### 第一步：拆解主题
将输入主题拆解为 3-5 个具体搜索角度。
例如主题"MCP server 最新实现"可拆解为：
- MCP server 开源项目（GitHub）
- MCP 协议规范（官方文档）
- MCP 在生产环境的实践案例（HN/Reddit）
- Claude Code 使用 MCP 的示例（X/社区）

### 第二步：多轮搜索
对每个角度执行 web_search，共 3-5 次搜索。
对最相关的 2-3 个结果执行 web_fetch 深度阅读。

### 第三步：综合分析
基于所有搜索结果，生成结构化研究报告，必须包含：
1. **核心发现**：最重要的 3-5 个信息点
2. **对 Building Agent 的具体价值**：可复用的架构/方法/工具
3. **推荐行动**：下一步应该做什么
4. **信息来源**：所有来源的 URL 列表

### 第四步：写入 Vault
1. 研究报告写入：
   D:\BuildingAgent_forobsidian\BuildingAgent\00_Inbox\research_[主题关键词]_[YYYY-MM-DD].md
2. 关键发现更新到对应 Wiki 笔记（追加，不覆盖）
3. 若无对应 Wiki 笔记，按模板新建

## 输出格式
```markdown
---
type: research_report
topic: [主题]
date: YYYY-MM-DD
sources_count: N
---

# 深度研究报告：[主题]

## 核心发现
1. ...
2. ...

## 对 Building Agent 的价值
- 可复用方法：...
- 可参考架构：...
- 可使用工具：...

## 推荐行动
- [ ] 立即：...
- [ ] 近期：...

## 信息来源
- [来源1](URL)
- [来源2](URL)
```
