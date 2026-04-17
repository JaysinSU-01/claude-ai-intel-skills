# /ai-classify-digest — 生成 Daily Digest + 更新 Wiki + 飞书推送

## 执行目标
读取过滤后的数据，生成 Daily Digest，更新 Wiki，推送飞书（如已配置）。

## 第一步：读取数据
读取最新的 D:\BuildingAgent_forobsidian\BuildingAgent\00_Inbox\filtered_*.md
读取 D:\BuildingAgent_forobsidian\BuildingAgent\99_Templates\daily_digest_template.md

## 第二步：确定 Top 3
从所有保留条目中选择 score 最高的 3 条作为 Top 3。
Top 3 的"产品视角"栏必须具体回答：
"该内容对 Building Agent（飞书多维表格 + 自然语言驱动的业务系统构建工具）的可复用价值是什么？"
具体说明可否转化为：subagent / skill / 功能模块 / 交互模式。

## 第三步：生成 Daily Digest
按 daily_digest_template.md 格式，在以下路径生成简报：
D:\BuildingAgent_forobsidian\BuildingAgent\01_Daily_Digest\[YYYY-MM-DD].md

填充规则：
- Top 3：仅从「🔴鲜活」层（< 14 天）中选 score 前 3，「近期参考」和「常青精选」不参与 Top 3 竞争
- 大厂动态：专门列出 Anthropic、OpenAI 相关条目（含所有时效层级）
- GitHub 趋势：列出所有来自 GitHub 且 score ≥ 3.0 的条目
- 各分类表格：「鲜活」条目在主表格，「近期参考」和「常青精选」在独立小节
- **每条条目必须包含 `发布时间：YYYY-MM-DD` 字段**
- 趋势总结：用 2-3 句话抽象描述本期最重要的变化方向（不重复具体条目内容）
- 建议跟进：从 Top 3 中提炼出值得 /ai-research 深入研究的方向

## 第四步：更新 Wiki
对每个 score ≥ 4.0 的条目：
1. 检查 02_Wiki/ 是否存在对应主题的 .md 文件
2. 若存在：在该笔记的"最新动态"表格中追加一行（不可覆盖已有内容）
3. 若不存在且主题重要：按 wiki_note_template.md 创建新笔记
4. 在 Daily Digest 中的关联 Wiki 字段填入 [[笔记名]]

## 第五步：同步飞书多维表格（仅在 feishu_config.json 已配置时执行）

### 5.1 读取配置
读取 D:\BuildingAgent_forobsidian\BuildingAgent\feishu_config.json
若文件不存在或 bitable_enabled=false，跳过整个第五步。

配置文件格式：
```json
{
  "app_id": "cli_xxxx",
  "app_secret": "xxxx",
  "bitable_app_token": "Nu82bwAUnaOUv7spXV3cza46neh",
  "bitable_table_id": "tblq8Qm6ksIOMFlt",
  "bitable_enabled": true,
  "webhook_url": "",
  "webhook_enabled": false
}
```

### 5.2 获取 tenant_access_token
```
POST https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal
Body: {"app_id": "...", "app_secret": "..."}
```
从响应中取 tenant_access_token，后续所有请求 Header 加 `Authorization: Bearer {token}`

### 5.3 确保多维表格列结构（首次运行时创建，后续跳过）
检查表格当前字段，若缺少以下字段则调用 POST /open-apis/bitable/v1/apps/{app_token}/tables/{table_id}/fields 创建：

| 字段名 | 类型 | 说明 |
|--------|------|------|
| 简报日期 | 日期（date） | 本次 digest 生成日期，格式 YYYY-MM-DD |
| 标题 | 文本（text） | 条目标题 |
| 来源链接 | 超链接（url） | 原文 URL |
| 分类 | 单选（single_select） | Agent Architecture / Prompt Engineering / Product Design / AI Infrastructure / Business Case |
| 时效层级 | 单选（single_select） | 🔴鲜活 / 🟡近期参考 / 🟢常青精选 |
| 热度分 | 数字（number） | 保留1位小数 |
| 平台 | 多选（multi_select） | GitHub / HN / Reddit / X / 大厂官方 |
| 发布时间 | 日期（date） | 原内容发布日期 |
| 一句话摘要 | 文本（text） | |
| 产品视角 | 多行文本（text） | 对 Building Agent 的价值 |
| 是否Top3 | 勾选（checkbox） | |
| 建议跟进 | 勾选（checkbox） | |

单选字段的选项颜色建议：
- 分类：Agent Architecture=蓝色、Prompt Engineering=绿色、Product Design=紫色、AI Infrastructure=橙色、Business Case=红色
- 时效层级：🔴鲜活=红色、🟡近期参考=黄色、🟢常青精选=绿色

### 5.4 批量写入记录
将本次 filtered_*.md 中所有保留条目，调用批量创建接口写入表格：
```
POST https://open.feishu.cn/open-apis/bitable/v1/apps/{app_token}/tables/{table_id}/records/batch_create
```
每次最多 500 条，超出则分批。字段值映射：
- `简报日期` → 本次运行日期（毫秒时间戳）
- `标题` → 条目标题文本
- `来源链接` → {text: 标题, link: URL}
- `分类` → 分类标签文本
- `时效层级` → 时效分层标签
- `热度分` → score 数值
- `平台` → 平台字段拆分为多选数组
- `发布时间` → published_at（毫秒时间戳，无法确认则留空）
- `一句话摘要` → 原始描述
- `产品视角` → 保留原因中的产品价值部分
- `是否Top3` → true/false
- `建议跟进` → true（若该条在「建议跟进」列表中）

### 5.5 写入完成后输出
输出：多维表格同步完成，写入 N 条记录 → {多维表格链接}

## 第六步：清理 Inbox
将 00_Inbox/ 下的 raw_*.md 和 filtered_*.md 移动至 04_Archive/[YYYY-MM]/
保持 00_Inbox/ 目录为空，等待下次 Scout 执行。

## 完成后
输出摘要：简报已生成 → 01_Daily_Digest/[日期].md | Wiki 更新 N 条 | 飞书推送：成功/跳过
