# CLAUDE.md

> 本文档为 Claude Code 和其他 AI 角色提供本项目的核心上下文。
> **所有 AI 在本项目工作时，必须首先阅读此文件**。

---

## 🎯 项目简介

**项目名**: `[项目全名]`  
**代号**: `[项目代号]`  
**类型**: `[Web 应用 / API 服务 / CLI 工具 / ...]`  
**当前阶段**: `[Stage 5 - 功能开发]`

**一句话描述**:
```
[一句话说清楚这个项目是什么]
```

**长描述**:
```
[2-3 段文字，描述项目目标、用户、核心价值]
```

---

## 🛠️ 技术栈速查

> 详细版本信息见 `.ai-workflow/04-architecture/TECH_STACK.md`

### 核心栈
- **语言**: TypeScript (strict mode)
- **Runtime**: Node.js >= 20
- **包管理**: pnpm

### 前端
- **框架**: Next.js 14 (App Router)
- **样式**: Tailwind CSS + [shadcn/ui]
- **状态**: [Zustand]
- **表单**: react-hook-form + Zod

### 后端
- **API**: Next.js API Routes
- **ORM**: Prisma
- **数据库**: PostgreSQL
- **缓存**: Redis

### 工具链
- **Linter**: ESLint
- **格式化**: Prettier
- **测试**: Vitest + Playwright
- **Git Hooks**: Husky

---

## 📁 目录结构

> 详见 `.ai-workflow/04-architecture/DIRECTORY_STRUCTURE.md`

**关键目录职责**:

```
src/
├── app/              ← Next.js 路由和 API
├── modules/          ← 业务模块（按领域组织）⭐ 主要工作区
│   ├── auth/         ← 认证模块
│   ├── user/         ← 用户模块（参考样板）
│   └── [其他]/
├── shared/           ← 跨模块共享（通用 UI、工具）
└── lib/              ← 基础设施（DB、缓存、日志）
```

**模块内部标准结构**:
```
modules/[name]/
├── components/       ← UI 组件
├── hooks/            ← React Hooks
├── services/         ← 业务逻辑
├── repositories/     ← 数据访问
├── types/            ← 类型定义
├── tests/            ← 测试
└── index.ts          ← 对外 API ⭐ 只通过这里暴露
```

---

## 📜 代码规范

> 详见 `.ai-workflow/00-core/CONSTRAINTS.md`

### 硬性要求 🔴

1. **TypeScript 严格模式** - 禁止 `any`
2. **参数化 SQL** - 防注入
3. **输入验证** - 所有用户输入用 Zod 验证
4. **错误处理** - 所有异步操作有 try-catch
5. **无硬编码敏感信息** - 用环境变量
6. **文档同步** - 架构变更必须同步文档

### 命名规范

- 变量/函数: `camelCase`
- 类/类型/接口: `PascalCase`
- 常量: `UPPER_SNAKE_CASE`
- 文件: 组件 `PascalCase.tsx`，其他 `camelCase.ts` 或 `kebab-case.ts`

### 禁用

- ❌ `any` 类型（除非必要且有注释）
- ❌ `console.log`（用 logger）
- ❌ 注释掉的代码
- ❌ 超过 50 行的函数（拆分）
- ❌ 跨模块访问内部文件（通过 index.ts）

---

## 🔑 常用命令

```bash
# 开发
pnpm dev                    # 启动开发服务器
pnpm build                  # 生产构建
pnpm start                  # 启动生产服务器

# 测试
pnpm test                   # 单元 + 集成测试
pnpm test:watch             # 监视模式
pnpm test:e2e               # E2E 测试
pnpm test:coverage          # 覆盖率报告

# 代码质量
pnpm lint                   # ESLint 检查
pnpm lint:fix               # 自动修复
pnpm format                 # Prettier 格式化
pnpm typecheck              # TypeScript 检查

# 数据库
pnpm db:migrate             # 运行迁移
pnpm db:studio              # 打开 Prisma Studio
pnpm db:seed                # 种子数据

# 依赖
pnpm install                # 安装依赖
pnpm audit                  # 漏洞扫描
```

---

## 🎭 核心模式

### 添加新功能的流程

```
1. 拿到任务文档 (.ai-workflow/06-development/features/FEAT-XXX.md)
2. 阅读任务 + 相关架构文档
3. 在对应 modules/[name]/ 下实现（不存在就新建）
4. 按分层顺序: Repository → Service → API → UI
5. 编写测试（与代码同步）
6. 自测验证所有验收标准
7. 提交 PR，经 R9 审查
8. 更新任务文档的"完成记录"
```

### 添加新模块的流程

```
1. 在 src/modules/ 下创建目录
2. 创建标准子目录（components/hooks/services/...）
3. 参考 modules/user/ 的结构
4. 编写 index.ts 只导出必要 API
5. 更新 .ai-workflow/00-core/GLOSSARY.md（如有新术语）
```

### 数据库迁移流程

```
1. 修改 prisma/schema.prisma
2. pnpm db:migrate dev --name [描述性名称]
3. 审查生成的 SQL
4. 在 staging 测试
5. 生产执行（备份后）
```

### API 新增端点流程

```
1. 参考 API_DESIGN.md 确定路径、方法
2. 在 src/app/api/v1/[路径]/route.ts 创建
3. Zod schema 验证输入
4. 调用 Service 层处理
5. 返回标准响应格式
6. 更新 API_DESIGN.md 端点清单
```

---

## 🚫 禁忌列表

### 绝对禁止 🚨
- ❌ 直接 push 到 `main` 分支
- ❌ 硬编码密钥/Token/密码
- ❌ 提交 `.env` 文件
- ❌ SQL 字符串拼接
- ❌ 日志中打印敏感信息
- ❌ 用户输入直接插入 HTML（XSS）
- ❌ 跳过权限检查

### 强烈避免 ⚠️
- ❌ 引入新的核心依赖（需决策）
- ❌ 偏离 DIRECTORY_STRUCTURE.md
- ❌ 写超长函数不拆分
- ❌ 忽略错误（吞掉异常）
- ❌ 做任务外的"顺手优化"

---

## 🔗 文档索引

### 核心文档 (必读)
- [项目宪法](./.ai-workflow/00-core/PROJECT_CONSTITUTION.md)
- [技术约束](./.ai-workflow/00-core/CONSTRAINTS.md)
- [决策记录](./.ai-workflow/00-core/DECISIONS.md)
- [术语表](./.ai-workflow/00-core/GLOSSARY.md)

### 架构文档 (开发必读)
- [架构总览](./.ai-workflow/04-architecture/ARCHITECTURE.md)
- [技术栈](./.ai-workflow/04-architecture/TECH_STACK.md)
- [数据库](./.ai-workflow/04-architecture/DATABASE.md)
- [API 设计](./.ai-workflow/04-architecture/API_DESIGN.md)
- [目录结构](./.ai-workflow/04-architecture/DIRECTORY_STRUCTURE.md)
- [安全设计](./.ai-workflow/04-architecture/SECURITY_DESIGN.md)

### 需求文档 (理解需求时读)
- [产品愿景](./.ai-workflow/02-vision/VISION.md)
- [需求规格](./.ai-workflow/03-requirements/REQUIREMENTS.md)
- [用户故事](./.ai-workflow/03-requirements/USER_STORIES.md)

### 工作流状态
- [当前状态](./.ai-workflow/meta/WORKFLOW_STATE.md)
- [待解决问题](./.ai-workflow/meta/OPEN_QUESTIONS.md)

---

## 🧭 AI 角色使用指南

### 当前在用的角色
- **R5 开发工程师**: 功能开发主力
- **R9 代码审查员**: 代码把关
- **R8 项目协调官**: 节奏把控（长期）

### 启动新角色时
1. 打开新的 Claude.ai 对话窗口
2. 复制 `.ai-workflow/prompts/` 下对应角色的 Prompt
3. 粘贴后 AI 会加载上下文（包括本文件）
4. 开始工作

---

## 💡 给 AI 的额外提示

### 你正在一个"严谨的工程项目"中工作

- ✅ 认真对待每个决策
- ✅ 怀疑时停下来问
- ✅ 优先保证质量，其次才是速度
- ✅ 写清楚的 commit message
- ✅ 更新文档不是负担，是责任

### 遇到以下情况请停下来询问

- 任务文档不清楚的地方
- 架构文档没覆盖的场景
- 约束之间似乎矛盾
- 需要引入新依赖
- 感觉当前任务与其他任务冲突

**停下来问 > 假设然后做错。**

---

## 📝 变更日志

| 日期 | 变更 | 变更者 |
|------|------|--------|
| YYYY-MM-DD | 初版 | R4 |
| | | |

---

> 🎯 **核心提醒**: 
> **本文件是 AI 角色的"项目入场券"**。保持它的最新，AI 才能靠谱工作。
> **每次重大变更（架构/约束/工具链）都要更新本文件。**
