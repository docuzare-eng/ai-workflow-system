# 项目术语表 (Glossary)

> 统一项目内的术语定义，避免歧义。**所有 AI 角色必须使用本文件中定义的标准术语。**
> 当发现对同一事物有多种称呼时，必须在此统一。

---

## 📖 使用说明

### 为什么需要术语表？
在复杂项目中，**同一概念的不同称呼**是混乱的主要来源：
- 一会儿叫 "用户"，一会儿叫 "客户"，一会儿叫 "会员"
- 一会儿叫 "订单"，一会儿叫 "交易"，一会儿叫 "记录"
- AI 角色之间传递信息时，术语不一致会导致严重误解

### 使用原则
1. **新概念首次出现**时，立即加入本表
2. **同义词合并**：选定一个标准术语，其他视为别名
3. **跨语言对照**：中英文都要有（代码中常用英文）
4. **保持单一真相源**：修改只在此处改，其他文档引用

---

## 一、业务核心概念 (Business Core Concepts)

> 项目最核心的领域概念，优先定义。

### [示例] 用户 (User)
- **定义**: 使用本产品的终端个体
- **英文**: User
- **代码命名**: `User`
- **相关概念**: 访客 (Visitor) / 会员 (Member)
- **区分**:
  - 访客 = 未注册的浏览者
  - 用户 = 已注册的个体
  - 会员 = 付费用户
- **常见误用**: 不要用 "Customer"（这个词留给 B2B 场景）

---
[术语名称] ([英文])

定义: [清晰、可操作的定义]
英文: [英文名]
代码命名: [代码中的标准写法]
相关概念: [相近的、容易混淆的概念]
区分: [与相关概念的边界]
常见误用: [不要怎么用]
示例: [具体例子]


> 💡 **请根据实际项目填充下方内容**

### [你的业务术语 1]
<!-- 按上方模板填写 -->

### [你的业务术语 2]
<!-- 按上方模板填写 -->

---

## 二、技术架构术语 (Technical Architecture)

> 架构层面的概念，由 R3 架构师定义。

### [示例] 服务 (Service)
- **定义**: 业务逻辑层的独立单元，封装一类相关功能
- **英文**: Service
- **代码命名**: `XxxService`
- **边界**: 一个 Service 对应一个业务领域，不对应 UI 或数据表
- **示例**: `UserService`、`OrderService`、`NotificationService`
- **区分**:
  - Service ≠ Controller（Controller 处理 HTTP 层）
  - Service ≠ Repository（Repository 处理数据访问）

### [技术术语 2]
<!-- 按模板填写 -->

---

## 三、产品功能术语 (Product Features)

> 产品层面的功能概念，由 R1/R2 定义。

### [示例] 工作流 (Workflow)
- **定义**: 用户完成某个目标的完整步骤链路
- **英文**: Workflow
- **区分于**:
  - 流程 (Process): 更偏向业务逻辑的执行
  - 任务 (Task): 工作流中的单个步骤
- **示例**: "7 阶段工作流" 指的是本系统的 Stage 0-7

### [功能术语 2]
<!-- -->

---

## 四、工作流术语 (Workflow Terminology)

> AI-Workflow-System 内部使用的术语。

### 阶段 (Stage)
- **定义**: 工作流的一个主要步骤（Stage 0-7）
- **英文**: Stage
- **区分**: Stage ≠ Phase（Phase 是更大的划分）

### 角色 (Role)
- **定义**: AI 扮演的专业身份，对应一套 Prompt 模板
- **英文**: Role
- **编号**: R0-R11

### 门禁 (Quality Gate)
- **定义**: 阶段间的质量检查点，未通过不能进入下一阶段
- **英文**: Quality Gate

### 交接笔记 (Handoff Note)
- **定义**: 阶段间传递的软信息，记录在 `CONTEXT_HANDOFF.md`
- **英文**: Handoff Note
- **区分于**: 产出物 (Deliverable)，后者是正式的文档

### 共享记忆层 (Shared Memory Layer)
- **定义**: `.ai-workflow/00-core/` 目录下的内容，所有角色共享
- **英文**: Shared Memory Layer
- **别名**: 项目基座、全局上下文

### 上下文漂移 (Context Drift)
- **定义**: 文档与实际代码/决策逐渐脱节的现象
- **英文**: Context Drift
- **应对**: R10 文档守护者定期检查

---

## 五、缩写表 (Abbreviations)

> 项目中使用的所有缩写必须在此注册。

| 缩写 | 全称 | 含义 |
|------|------|------|
| ADR | Architecture Decision Record | 架构决策记录 |
| PRD | Product Requirements Document | 产品需求文档 |
| MVP | Minimum Viable Product | 最小可用产品 |
| POC | Proof of Concept | 概念验证 |
| DX | Developer Experience | 开发者体验 |
| UX | User Experience | 用户体验 |
| UI | User Interface | 用户界面 |
| API | Application Programming Interface | 应用编程接口 |
| CI/CD | Continuous Integration / Continuous Deployment | 持续集成/持续部署 |
| LCP | Largest Contentful Paint | 最大内容绘制 (性能指标) |
| INP | Interaction to Next Paint | 交互响应 (性能指标) |
| SLA | Service Level Agreement | 服务等级协议 |
| SLO | Service Level Objective | 服务等级目标 |

> 💡 项目特有的缩写请追加至此表。

---

## 六、容易混淆的词对 (Commonly Confused Pairs)

> 明确区分容易混用的词。

### 用户 (User) vs 客户 (Customer)
- **User**: 使用产品的个体（重点在"使用"）
- **Customer**: 付费的个体或组织（重点在"付费"）
- **使用规则**: B2C 产品用 User，B2B 产品用 Customer

### 请求 (Request) vs 查询 (Query)
- **Request**: HTTP 请求或其他网络调用
- **Query**: 数据查询（数据库查询、GraphQL 查询）

### 创建 (Create) vs 新增 (Add)
- **Create**: 从无到有创建新实体 (`createUser`)
- **Add**: 向已存在的集合添加元素 (`addItemToCart`)

### 删除 (Delete) vs 移除 (Remove)
- **Delete**: 永久删除，数据消失 (`deleteUser`)
- **Remove**: 从集合中移除引用，实体可能还在 (`removeItemFromCart`)

> 💡 项目中发现其他容易混淆的词对，追加至此。

---

## 七、外部术语对照 (External References)

> 当项目涉及特定领域或引用外部标准时，在此对照说明。

### [示例] 本项目术语 vs RESTful API 规范
- 我们的 "资源" = REST 中的 "Resource"
- 我们的 "操作" = REST 中的 "Method"

---

## 更新日志

| 日期 | 变更内容 | 变更者 |
|------|---------|-------|
| YYYY-MM-DD | 文档创建 | - |
| | | |

---

## 🚨 术语使用规则（强制）

1. **新概念首次出现**: 必须在此注册，否则产出视为不规范
2. **修改已有术语**: 必须通知所有角色，并检查全项目一致性
3. **跨文档一致**: 禁止在不同文档中使用不同称呼
4. **代码与文档一致**: 代码中的类名、变量名要与术语表对应

---

> 📌 **核心提醒**: 术语表看似琐碎，却是**大项目不崩盘**的关键。
> 当一个概念有两种叫法时，就埋下了一颗定时炸弹。
> 花 5 分钟注册术语，省下未来 5 小时的返工。
