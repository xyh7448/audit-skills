# 项目代码审计官 Skills

一套标准化的 AI Agent 审计技能集，让 Agent 按照固定方法论自动检查系统质量。

## 为什么需要这套 Skills？

大多数人在让 AI Agent 写代码，但真正缺的不是编码能力，而是一套**标准化审计体系**。

有了这套 Skills，Agent 会自动完成：

```
开发 → 审计 → 对齐 → 验证数据源 → 修复 → CTO评审 → 上线
```

## 6 个命令

| 命令 | 用途 | 使用时机 |
|------|------|---------|
| `/audit-feat <模块名>` | 功能全景审计（八维评分） | 开发完一个功能后 |
| `/audit-febe <模块名>` | 前后端对齐审计（API/字段/类型） | 联调或发现数据不对时 |
| `/audit-data <模块名>` | 数据源真实性审计（逐字段溯源） | 发现数据可疑时 |
| `/audit-prod <模块名>` | 生产就绪审计（0-105 分量化） | 上线前 |
| `/cto <模块名>` | CTO 终审（A/B/C 等级判定） | 每次开发结束后 |
| `/fix <模块名>` | 规划修复 → 执行 → 验收 | 审计发现问题后 |

## 推荐执行顺序

```
/audit-feat → /audit-febe → /audit-data → /audit-prod → /fix → /cto
```

## 特性

- **框架无关**：适用于 FastAPI、Express、Django、Next.js、Spring Boot 等任何技术栈
- **自动发现项目约定**：审计前自动扫描 AGENTS.md、CONVENTIONS.md 等项目规则文件
- **量化评分**：每个审计都有明确的评分标准和输出格式
- **闭环修复**：/fix 提供从问题收集到验收测试的完整修复工作流
- **CTO 终审**：强制前置审计，数据造假一票否决

## 各 Skill 详解

### `/audit-feat` — 功能全景审计

八维审计：功能完整性、架构合理性、数据流、API 层、服务层、数据库层、异常处理、性能问题。

### `/audit-febe` — 前后端对齐审计

逐项核对：API 接口、字段一致性、类型一致性、服务端渲染 Context、WebSocket 消息、Mock 检测。

### `/audit-data` — 数据源真实性审计

逐字段反向追踪：用户看到 → 前端变量 → API → Service → 数据获取层 → 最终数据源。标记 FAKE DATA。

### `/audit-prod` — 生产就绪审计

11 个维度：安全、权限、缓存、日志、限流、异常、数据库、部署、环境变量、成本、异步安全。总分 105。

### `/cto` — CTO 终审

七维评估：用户价值、数据真实性（一票否决）、技术架构、扩展能力、维护成本、商业可行性、风险。

### `/fix` — 审计问题修复

四阶段闭环：收集问题 → 规划修复（必须用户确认）→ 逐项修复 → 验收测试。

## 安装

```bash
# 作为项目级 Skills（推荐）
cp -r audit-feat audit-febe audit-data audit-prod cto fix .qoder/skills/

# 或作为个人全局 Skills
cp -r audit-feat audit-febe audit-data audit-prod cto fix ~/.qoder/skills/
```

## 目录结构

```
qoder-audit-skills/
├── README.md
├── audit-feat/SKILL.md    # 功能全景审计
├── audit-febe/SKILL.md    # 前后端对齐审计
├── audit-data/SKILL.md    # 数据源真实性审计
├── audit-prod/SKILL.md    # 生产就绪审计
├── cto/SKILL.md           # CTO 终审
└── fix/SKILL.md           # 修复工作流
```

## License

MIT
