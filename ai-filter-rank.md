# /ai-filter-rank — 过滤、评分、去重

## 执行目标
读取 00_Inbox 中最新的 raw_*.md 文件，按 Core Brain 规则过滤，并评分排序。

## 第一步：读取数据
读取 D:\BuildingAgent_forobsidian\BuildingAgent\00_Inbox\ 目录下最新的 raw_*.md 文件。
读取 D:\BuildingAgent_forobsidian\BuildingAgent\weights.json 获取当前权重。

## 第二步：过滤规则（强制执行，每条必须满足至少一项才保留）

### 保留条件（至少满足一项）
1. **技术创新**：涉及新架构（multi-agent、MCP等）、新能力（memory、tool use、computer use等）
2. **产品启发**：新交互方式、新工作流设计、新系统结构
3. **趋势信号**：多来源重复出现、大厂发布、GitHub 快速增长（今日新增 star >100）
4. **可复用方法**：可提炼为 prompt 技巧、agent 模式、workflow 结构

### 强制删除（满足任何一项则删除）
1. 无实质技术内容的新闻转载（只有事件描述，无方法/架构/数据）
2. 纯观点讨论或情绪表达（无数据支撑、无可提炼方法）
3. 标题党（标题与内容严重不符）
4. 无法提炼核心观点的内容
5. 与 AI/Agent/Building Agent 无关的内容

## 第二点五步：时效分层（评分前强制执行）

根据内容发布时间，将每条内容分入以下三层，层级决定后续处理策略：

| 层级 | 发布时间 | 进入 Top 3？ | 最低保留 hotness |
|------|----------|-------------|-----------------|
| 🔴 鲜活（Fresh） | < 14 天 | ✅ 可以 | 无额外要求 |
| 🟡 近期参考（Recent） | 14 ~ 90 天 | ❌ 不可以 | hotness ≥ 3.5 |
| 🟢 常青精选（Evergreen） | > 90 天（含所有 2025 年内容） | ❌ 不可以 | hotness ≥ 4.0 |

- 层级标注附加到每条内容的标签中，如 `[score=3.2] [🟡近期参考] [Agent Architecture]`
- 「近期参考」和「常青精选」在 Daily Digest 中单独成一节，不占主体表格位置

## 第三步：评分公式

```
score = (hotness × 0.6) + (freshness × 0.4)
```

### hotness 计算（0-5 分）
- GitHub 新增 star >500/天：5分 | 200-500：4分 | 50-200：3分 | <50：2分
- HN points >300：5分 | 100-300：4分 | 30-100：3分 | <30：2分
- Reddit 评论 >100：5分 | 50-100：4分 | 10-50：3分 | <10：2分
- X/Twitter 互动高（估算）：3-5分 | 低：1-2分
- 大厂官方发布：直接 5分（固定）

### freshness 计算（0-2 分）
- 发布时间 <24小时：+2
- 24~48小时：+1
- 3~14天：0
- 14~90天：-0.5（下限 0）
- >90天：-1（下限 0）

### 权重调整
最终 score 再乘以该条目所属类别的权重（来自 weights.json）

## 第四步：分类（每条必须打一个标签）
- **Agent Architecture**：框架、架构、multi-agent、MCP、tool use
- **Prompt Engineering**：prompt 技巧、system prompt、AGENTS.md、CoT
- **Product Design**：交互、UX、工作流、应用产品
- **AI Infrastructure**：模型、训练、基础设施、API
- **Business Case**：商业应用、竞品、市场案例

### 类别 ID 与 Building Agent 关联程度
优先保留与 Agent Architecture、Prompt Engineering、Product Design 强相关的内容。

## 第五步：去重规则
- 同一 URL → 合并（保留，不重复）
- 不同平台同一主题 → 保留最高 score 的那条，其他平台信息合并到该条的"平台"字段
- score < 1.5 的条目 → 直接丢弃

## 输出格式
将评分结果写入：
D:\BuildingAgent_forobsidian\BuildingAgent\00_Inbox\filtered_[YYYY-MM-DD].md

格式：
```
---
filter_date: YYYY-MM-DD
input_count: N
output_count: N
discarded_count: N
fresh_count: N
recent_count: N
evergreen_count: N
---

# Filtered & Ranked - YYYY-MM-DD

## 🔴 鲜活内容（< 14 天，可进 Top 3）

### [score=4.8] [Agent Architecture] [标题](URL)
- 平台：GitHub | 发布时间：YYYY-MM-DD | 距今：N天
- 原始描述：xxx
- 保留原因：技术创新 - 新架构

## 🟡 近期参考（14~90 天，不进 Top 3）

### [score=3.6] [Prompt Engineering] [标题](URL)
- 平台：HN | 发布时间：YYYY-MM-DD | 距今：N天
- 原始描述：xxx
- 保留原因：可复用方法

## 🟢 常青精选（> 90 天，不进 Top 3）

### [score=4.2] [Agent Architecture] [标题](URL)
- 平台：HN | 发布时间：YYYY-MM-DD | 距今：N天
- 原始描述：xxx
- 保留原因：高分经典参考

## 丢弃条目摘要（仅记录丢弃原因，不保留原始内容）
- 共丢弃 N 条：纯观点N条 / 无关N条 / 过低分N条 / 时效不足N条
```

## 完成后
输出摘要：过滤完成，原始 N 条 → 保留 N 条（丢弃 N 条），已写入 filtered_[日期].md。
