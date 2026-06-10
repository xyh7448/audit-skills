---
name: audit-febe
description: "前后端对齐审计（FE+BE）。用法：/audit-febe <模块名>，如 /audit-febe export-assessment。检查API接口、字段一致性、类型一致性、页面数据来源、Mock检测。联调或发现数据不对时执行。"
---

# 前后端对齐审计（Frontend-Backend Alignment）

你是一名前后端集成审计专家。请对指定功能模块的前后端对齐状态进行全面检查。

## 触发方式

```
/audit-febe <module-name>
```

示例：`/audit-febe export-assessment`、`/audit-febe chat`

## 审计流程

### 第一步：收集后端 API 清单

1. 用 `Grep` 搜索 `app/routers/` 下模块相关路由文件
2. 用 `SearchSymbol` 追踪每个路由函数的参数 Schema 和返回类型
3. 用 `Grep` 在 `app/main.py` 检查路由是否已挂载（`include_router`）
4. 提取每个 API 的：URL、Method、参数 Schema（Pydantic）、返回 Schema

### 第二步：收集前端调用清单

1. 用 `Grep` 搜索 `templates/` 中的 `fetch\(|axios\.|\$.ajax|new WebSocket|\.send\(`
2. 用 `Grep` 搜索 `static/js/` 和 `static/report-dashboard/` 中的 API 调用
3. 用 `Grep` 搜索 Jinja2 模板中的 `render_template\(` 调用，追踪传入的 context 变量
4. 用 `Grep` 搜索 WebSocket 消息格式（`type.*payload|action.*data`）
5. 提取每个前端调用的：URL、Method、发送参数、期望返回

### 第三步：逐项核对

#### 1. API 接口对齐

逐个核对前端调用 ↔ 后端实现：

| 检查项 | 方法 |
|--------|------|
| URL 匹配 | 前端请求路径 = 后端注册路径 |
| Method 匹配 | GET/POST/PUT/DELETE 一致 |
| 参数匹配 | Query/Body/Path 参数名和类型一致 |
| 返回格式 | 前端解析方式 = 后端返回结构 |

#### 2. 字段一致性

对比前端需要的字段 VS 后端实际返回的字段：

- **缺失字段**：前端用到但后端没返回
- **多余字段**：后端返回但前端没用
- **命名不一致**：同一数据前后端字段名不同（如 `hsCode` vs `hs_code`）

方法：追踪后端 Pydantic Response Schema → 前端模板/JS 中的字段引用

#### 3. 类型一致性

检查每个字段的前后端类型是否匹配：

| 后端类型 | 前端期望 | 风险 |
|---------|---------|------|
| `str` | `number` | 高 - 计算错误 |
| `list` | `string` | 高 - 渲染崩溃 |
| `Optional[T]` | 非 null | 高 - 空指针 |
| `datetime` | `string` | 中 - 格式解析 |

#### 4. 页面展示验证

逐个页面检查：

1. 找到页面模板文件
2. 列出页面展示的所有数据字段
3. 追溯每个字段的数据来源（API → Service → DB/外部API）
4. 标记无法追溯到真实来源的字段

#### 5. Mock 检测

使用以下关键词搜索模块相关文件：

```
mock, fake, hardcode, placeholder, dummy, stub, sample, test_data,
TODO, FIXME, temporary, 临时, 模拟, 假数据
```

分类标记：
- **硬编码**：直接写死的值
- **Mock**：函数返回固定值
- **Placeholder**：占位文本（"暂无数据"、"N/A"）
- **Sample**：示例数据未替换为真实数据

## 输出格式

```markdown
# [模块名] 前后端对齐审计报告

## API 对齐总览
| # | API | 前端调用 | 后端实现 | 状态 |
|---|-----|---------|---------|------|
| 1 | GET /api/xxx | ✅ 已调用 | ✅ 已实现 | 已对齐 |
| 2 | POST /api/yyy | ❌ 未调用 | ✅ 已实现 | 前端未接入 |
| 3 | GET /api/zzz | ✅ 已调用 | ❌ 未实现 | 后端缺失 |

## 已对齐 ✅
[列表]

## 未对齐 ❌
[列表 + 具体问题描述]

## 字段差异明细
| 字段 | 前端期望 | 后端实际 | 差异类型 |
|------|---------|---------|---------|

## Mock / 假数据清单
| 文件 | 行号 | 内容 | 类型 | 严重程度 |
|------|------|------|------|---------|

## 风险等级
- 🔴 高风险：[数量] 项（数据不可用/类型不匹配导致崩溃）
- 🟡 中风险：[数量] 项（数据不完整但不影响核心流程）
- 🟢 低风险：[数量] 项（命名不规范等）

## 修复方案
[按优先级排列的具体修复步骤]
```

## 注意事项

- **Jinja2 Context 追踪**：用 `Grep` 搜索 `render_template.*模板名`，检查传入的 context dict 是否包含模板中所有 `{{ variable }}` 引用
- 重点关注 WebSocket 消息的字段对齐（chat 模块）——用 `Grep` 搜索 `json\.dumps.*type|send_text|send_json` 追踪 WS 消息格式
- 检查 `static/report-dashboard/` 的 React 组件 API 调用
- 前端没有 TypeScript 时，通过 JS 代码中的字段访问推断期望类型
- 用 `SearchSymbol` 追踪后端 Pydantic Response Schema 的字段定义
