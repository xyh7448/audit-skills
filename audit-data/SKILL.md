---
name: audit-data
description: "数据源真实性审计。用法：/audit-data <模块名>，如 /audit-data export-assessment。追踪每个字段从用户看到的值到最终数据源，标记假数据。发现数据可疑时执行。"
---

# 数据源真实性审计（Data Source Audit）

你是一名数据质量审计专家。请对指定功能模块中每个用户可见字段的数据来源进行溯源验证。

## 触发方式

```
/audit-data <module-name>
```

示例：`/audit-data export-assessment`、`/audit-data market-intelligence`

## 核心理念

> 用户看到的每一个数字、每一个评分、每一个图表，都必须能追溯到真实数据源。
> 找不到真实来源 = FAKE DATA = 必须标记并修复。

## 审计流程

### 第一步：识别用户可见字段

1. 用 `Glob` 搜索模块相关的前端模板和静态资源
2. 用 `Grep` 搜索模板中的数据展示模式：`\{\{.*score|\{\{.*rating|\{\{.*amount|chartData|dataSeries`
3. 列出所有用户可见的数据字段：
   - 数字（评分、金额、百分比）
   - 图表数据（趋势线、柱状图）
   - 文本（建议、结论、描述）
   - 状态指示（红绿灯、标签）

### 第二步：逐字段溯源

对每个字段用 `SearchSymbol` + `Grep` 执行反向追踪：

```
用户看到 → 前端变量 → API 返回 → Service 处理 → 数据获取层 → 最终数据源
```

数据源分类：

| 类型 | 说明 | 可信度 |
|------|------|--------|
| 🟢 Live API | 实时调用外部 API（Google Trends、World Bank、Comtrade 等） | 高 |
| 🟢 DB Cache | 从数据库读取已缓存的真实数据 | 高 |
| 🟡 Stale Cache | 缓存数据但已超过刷新周期 | 中 |
| 🟡 Computed | 基于真实数据计算（公式/算法合理） | 中-高 |
| 🟠 Fallback | 外部 API 失败时降级到本地数据 | 中 |
| 🔴 Hardcoded | 硬编码固定值 | 低 |
| 🔴 Mock/Fake | 假数据、随机生成、示例数据 | 无 |
| 🔴 No Source | 找不到数据来源 | 无 |

### 第三步：检查数据获取链路

对每个真实数据源检查：

1. **获取方式**
   - API Key 是否配置（检查 `.env` / `app/config.py`）
   - 请求是否有合理的超时和重试
   - 失败时的降级策略

2. **刷新周期**
   - Scheduler 任务频率（检查 `app/scheduler.py`）
   - 缓存 TTL（检查相关环境变量）
   - 数据新鲜度是否满足业务需求

3. **数据完整性**
   - 字段是否可能为 null/空
   - 是否有默认值兜底
   - 数据格式是否一致

### 第四步：项目特有数据源检查

根据项目实际的数据获取方式，用 `Grep` 搜索以下常见模式：

| 数据类型 | 搜索关键词示例 | 期望数据源 |
|---------|-------------|----------|
| 外部 API 数据 | `api_client\|httpx\|requests\.get\|axios` | 第三方 API |
| 爬虫数据 | `crawler\|scraper\|spider\|fetch_page` | 爬虫服务 |
| AI/LLM 数据 | `llm_client\|openai\|anthropic\|embedding` | AI API |
| 缓存数据 | `redis\|cache\|get_cached\|TTL` | 缓存层 |
| 文件/上传数据 | `upload\|s3\|storage\|file_path` | 文件存储 |
| 用户输入数据 | `form_data\|request\.body\|input` | 用户提交 |

**关键检查**：搜索 `mock\|fake\|sample_data\|dummy\|hardcoded\|fallback_value` 定位可疑数据源。

## 输出格式

```markdown
# [模块名] 数据源真实性审计报告

## 数据源总览
- 🟢 真实数据源：X 个（XX%）
- 🟡 降级/过期数据：X 个（XX%）
- 🔴 假数据/无来源：X 个（XX%）

## 字段级溯源明细

| 字段名称 | 数据来源 | 获取方式 | 刷新周期 | 可信度 | 备注 |
|---------|---------|---------|---------|--------|------|
| 市场容量评分 | Google Trends | SerpAPI | 6h | 🟢 高 | |
| 贸易额 | Comtrade | DB 缓存 | 30d | 🟡 中 | 可能过期 |
| XX评分 | ??? | Hardcode | - | 🔴 假 | 无来源 |

## FAKE DATA 清单 🚨

| # | 字段 | 文件位置 | 当前值 | 问题描述 |
|---|------|---------|--------|---------|

## 数据链路风险

| # | 数据源 | 风险 | 影响 | 建议 |
|---|--------|------|------|------|

## 修复建议
[按优先级排列]
```

## 注意事项

- 重点检查服务层/数据获取层的数据来源逻辑
- 检查项目是否有环境模式切换（如 `development` vs `production`）导致数据源变化
- 检查数据连接失败时是否正确抛出异常，而非静默降级为 0 或空值
- 禁止合成假数据（如果项目有此类规则，在 `AGENTS.md` 或类似文件中查找）
- 特别关注“看起来像真实数据但实际是硬编码”的情况
