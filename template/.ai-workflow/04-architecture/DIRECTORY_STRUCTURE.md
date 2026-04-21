# 目录结构设计 (Directory Structure)

> 本文件定义项目的完整目录结构，是 R4 工程骨架师搭建项目的依据。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| 技术栈 | `[如 Next.js 14 + TypeScript]` |
| 架构模式 | `[如 模块化单体]` |
| 文档版本 | `v1.0` |
| 最后更新 | `YYYY-MM-DD` |

---

## 一、组织原则

### 1.1 核心原则

- **按领域组织** > **按类型组织**
  - ✅ `modules/user/{components,services,types}` 
  - ❌ `components/user/*, services/user/*, types/user/*`

- **高内聚低耦合**
  - 相关的文件放一起
  - 模块间通过明确 API 通信

- **可预测性**
  - 新增某类文件时，位置应该显而易见
  - 新人能在 30 分钟内理解结构

### 1.2 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 目录 | `kebab-case` | `user-profile/` |
| React 组件 | `PascalCase.tsx` | `UserCard.tsx` |
| 非组件 TS 文件 | `camelCase.ts` | `userService.ts` 或 `user.service.ts` |
| 测试文件 | `*.test.ts` / `*.spec.ts` | `user.service.test.ts` |
| 常量文件 | `constants.ts` | - |
| 类型文件 | `types.ts` 或 `*.types.ts` | - |

---

## 二、完整目录树

```
[project-name]/
│
├── .ai-workflow/                  # 工作流元数据（本系统）
│   ├── 00-core/
│   ├── 01-feasibility/
│   ├── ... (其他阶段)
│   ├── meta/
│   └── logs/
│
├── .github/                       # GitHub 配置
│   ├── workflows/                 # CI/CD
│   │   ├── ci.yml
│   │   └── deploy.yml
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
│
├── .husky/                        # Git Hooks
│   ├── pre-commit
│   └── commit-msg
│
├── .vscode/                       # 编辑器配置（可选）
│   ├── settings.json
│   └── extensions.json
│
├── public/                        # 静态资源
│   ├── favicon.ico
│   └── images/
│
├── src/                           # 源代码（主目录）
│   │
│   ├── app/                       # Next.js App Router
│   │   ├── (public)/             # 公开页面路由组
│   │   │   ├── page.tsx          # 首页
│   │   │   ├── about/
│   │   │   └── layout.tsx
│   │   ├── (auth)/               # 认证页面路由组
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── layout.tsx
│   │   ├── (protected)/          # 需登录页面
│   │   │   ├── dashboard/
│   │   │   ├── settings/
│   │   │   └── layout.tsx
│   │   ├── api/                  # API 路由
│   │   │   └── v1/
│   │   │       ├── auth/
│   │   │       ├── users/
│   │   │       └── [其他]/
│   │   ├── layout.tsx            # 根布局
│   │   ├── error.tsx             # 全局错误页
│   │   ├── not-found.tsx         # 404 页
│   │   └── globals.css
│   │
│   ├── modules/                   # 业务模块（核心）
│   │   │
│   │   ├── auth/                  # 认证模块
│   │   │   ├── components/       # 模块内 UI 组件
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   └── RegisterForm.tsx
│   │   │   ├── hooks/            # 自定义 Hooks
│   │   │   │   └── useAuth.ts
│   │   │   ├── services/         # 业务服务
│   │   │   │   └── auth.service.ts
│   │   │   ├── repositories/     # 数据访问
│   │   │   │   └── auth.repository.ts
│   │   │   ├── types/            # 类型定义
│   │   │   │   └── auth.types.ts
│   │   │   ├── utils/            # 模块内工具
│   │   │   │   └── password.ts
│   │   │   ├── tests/            # 模块测试
│   │   │   │   ├── auth.service.test.ts
│   │   │   │   └── __mocks__/
│   │   │   └── index.ts          # 对外 API
│   │   │
│   │   ├── user/                  # 用户模块
│   │   │   └── ... (同上结构)
│   │   │
│   │   ├── [domain1]/             # 核心业务域 1
│   │   │   └── ...
│   │   │
│   │   └── [domain2]/             # 核心业务域 2
│   │       └── ...
│   │
│   ├── shared/                    # 跨模块共享
│   │   ├── components/           # 通用 UI 组件
│   │   │   ├── ui/              # 基础 UI（按钮、输入框等）
│   │   │   └── layout/          # 布局组件
│   │   ├── hooks/                # 通用 Hooks
│   │   ├── utils/                # 通用工具
│   │   ├── types/                # 通用类型
│   │   └── constants/            # 常量
│   │
│   ├── lib/                       # 基础设施
│   │   ├── db/                   # 数据库
│   │   │   ├── client.ts         # DB 客户端
│   │   │   └── migrations/       # 迁移文件（如 Prisma）
│   │   ├── auth/                 # 认证基础设施
│   │   │   └── jwt.ts
│   │   ├── cache/                # 缓存
│   │   │   └── redis.ts
│   │   ├── email/                # 邮件服务
│   │   │   └── client.ts
│   │   ├── logger/               # 日志
│   │   │   └── logger.ts
│   │   ├── monitoring/           # 监控
│   │   └── env.ts                # 环境变量（类型化 + 验证）
│   │
│   ├── middleware.ts              # Next.js 中间件
│   ├── config/                    # 配置
│   │   └── constants.ts
│   │
│   └── styles/                    # 样式
│       └── themes/
│
├── prisma/                        # Prisma ORM（如使用）
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
│
├── tests/                         # 端到端测试（E2E）
│   ├── e2e/
│   │   ├── auth.spec.ts
│   │   └── ...
│   ├── fixtures/                 # 测试数据
│   └── playwright.config.ts
│
├── scripts/                       # 脚本
│   ├── seed.ts                   # 种子数据
│   ├── backup.ts                 # 备份
│   └── migration/                # 数据迁移脚本
│
├── docs/                          # 额外文档（非工作流）
│   ├── api/                      # API 文档
│   ├── guides/                   # 操作指南
│   └── ARCHITECTURE.md           # 架构速查（可链接 .ai-workflow）
│
├── .env.example                   # 环境变量示例
├── .env.local                     # 本地环境变量（.gitignore）
├── .gitignore
├── .eslintrc.json
├── .prettierrc
├── .editorconfig
├── commitlint.config.js
├── next.config.js                 # Next.js 配置
├── tailwind.config.ts             # Tailwind 配置
├── tsconfig.json                  # TypeScript 配置
├── vitest.config.ts               # 测试配置
├── package.json
├── pnpm-lock.yaml
├── README.md
├── CLAUDE.md                      # Claude Code 上下文（关键！）
├── CONTRIBUTING.md
├── CHANGELOG.md
└── LICENSE
```

---

## 三、目录职责详解

### 3.1 `src/app/` - 路由层

**职责**:
- Next.js App Router 路由
- 页面组件（`page.tsx`）
- 布局（`layout.tsx`）
- API 路由

**不放**:
- ❌ 业务逻辑（去 `modules/`）
- ❌ 可复用组件（去 `modules/*/components` 或 `shared/components`）

**路由组说明**:
- `(public)/`: 公开页面
- `(auth)/`: 登录注册等
- `(protected)/`: 需要登录的页面

### 3.2 `src/modules/` - 业务模块层（核心）

**职责**: 按业务领域组织代码

**结构规则**: 每个模块遵循相同结构
```
[moduleName]/
├── components/    ← UI 组件（仅本模块用）
├── hooks/         ← React Hooks
├── services/      ← 业务逻辑
├── repositories/  ← 数据访问
├── types/         ← 类型
├── utils/         ← 模块工具
├── tests/         ← 测试
└── index.ts       ← 对外 API（仅导出必要的）
```

**关键规则**:
- ✅ 模块内**高内聚**
- ✅ 通过 `index.ts` 暴露 API
- ❌ 不跨模块访问内部文件
- ❌ 不循环依赖

**对外暴露示例** (`modules/user/index.ts`):
```typescript
// ✅ 暴露必要的 API
export { UserService } from './services/user.service';
export type { User, UserProfile } from './types/user.types';

// ❌ 不暴露内部实现
// export * from './repositories/user.repository'; 这是错的
```

### 3.3 `src/shared/` - 共享层

**职责**: 跨模块共享的"与业务无关"的东西

**放什么**:
- ✅ 通用 UI 组件（按钮、输入框、Modal）
- ✅ 通用 Hooks（useLocalStorage 等）
- ✅ 纯工具函数
- ✅ 通用类型

**不放什么**:
- ❌ 业务相关的（应该放 `modules`）

### 3.4 `src/lib/` - 基础设施层

**职责**: 技术性、与业务无关的基础设施

**放什么**:
- ✅ 数据库客户端
- ✅ 缓存客户端
- ✅ 日志配置
- ✅ 邮件/短信客户端
- ✅ 监控集成

**特点**: 
- 业务模块**通过抽象依赖**这些（不直接调用具体实现）
- 未来可替换（如换数据库）不影响业务

### 3.5 `src/middleware.ts` - 全局中间件

**职责**: 跨路由的横切逻辑
- 认证检查
- 请求日志
- Rate limiting
- CORS

### 3.6 `prisma/` - ORM Schema（如使用 Prisma）

- `schema.prisma`: 数据模型定义
- `migrations/`: 迁移文件（自动生成）
- `seed.ts`: 种子数据

### 3.7 `tests/` - E2E 测试

**与 `modules/*/tests/` 的区别**:
- `modules/*/tests/`: 单元测试、集成测试（模块内）
- `tests/e2e/`: 端到端测试（跨模块、模拟真实用户）

---

## 四、文件放置决策树

### 我该把新文件放哪里？

```
是否是路由/页面？
  └─ 是 → src/app/...
  └─ 否 ↓

是 UI 组件？
  ├─ 只服务于一个模块 → modules/[module]/components/
  ├─ 跨模块通用 → shared/components/
  └─ 不是 UI 组件 ↓

是业务逻辑？
  └─ 是 → modules/[对应模块]/services/
  └─ 否 ↓

是数据访问？
  └─ 是 → modules/[对应模块]/repositories/
  └─ 否 ↓

是基础设施（数据库、缓存等）？
  └─ 是 → lib/
  └─ 否 ↓

是通用工具？
  └─ 是 → shared/utils/
  └─ 否 ↓

不确定？
  └─ 问 R3 或 R8 协调官
```

---

## 五、反模式（要避免）

### ❌ 反模式 1: "按类型组织"

```
components/
  user/*
  post/*
services/
  user/*
  post/*
```

**问题**: 改一个功能要动 N 个目录

### ❌ 反模式 2: 巨型模块

```
modules/core/* (里面 50 个不相关的东西)
```

**问题**: 失去模块化的意义

### ❌ 反模式 3: 跨模块内部访问

```typescript
// ❌ 错误：直接访问 user 模块内部
import { UserRepository } from '@/modules/user/repositories/user.repository';

// ✅ 正确：通过对外 API
import { UserService } from '@/modules/user';
```

### ❌ 反模式 4: 业务代码放 lib/

```
lib/userService.ts  ❌ 业务代码不该在这
```

---

## 六、导入路径约定

### TypeScript 路径别名

在 `tsconfig.json` 配置：

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/modules/*": ["./src/modules/*"],
      "@/shared/*": ["./src/shared/*"],
      "@/lib/*": ["./src/lib/*"]
    }
  }
}
```

### 使用示例

```typescript
// ✅ 清晰、可重构
import { UserService } from '@/modules/user';
import { Button } from '@/shared/components/ui/Button';
import { db } from '@/lib/db/client';

// ❌ 避免相对路径爬升
import { UserService } from '../../../modules/user';
```

---

## 七、新增模块的步骤

```
1. 在 src/modules/ 下创建目录
   mkdir src/modules/[new-module]

2. 创建标准子目录
   cd src/modules/[new-module]
   mkdir components hooks services repositories types utils tests

3. 参考 modules/user/ 的 index.ts 创建对外 API
   touch index.ts

4. 添加第一个类型定义
   touch types/[new-module].types.ts

5. 添加第一个服务
   touch services/[new-module].service.ts

6. 添加第一个测试
   touch tests/[new-module].service.test.ts

7. 更新 .ai-workflow/00-core/GLOSSARY.md（如有新术语）
```

---

## 八、特殊目录说明

### `.ai-workflow/` - 不要修改（工作流专用）

这是本工作流系统的元数据，按规范使用即可。

### `scripts/` - 运维脚本

- 一次性脚本：加注释说明用途
- 定期运行脚本：加到 cron 或 CI

### `public/` - 静态资源

- 可通过 URL 直接访问
- 不要放敏感文件

---

## 九、配置文件说明

| 文件 | 用途 |
|------|------|
| `.env.example` | 环境变量模板（提交到 Git） |
| `.env.local` | 本地实际值（不提交） |
| `tsconfig.json` | TypeScript 配置 |
| `next.config.js` | Next.js 配置 |
| `tailwind.config.ts` | Tailwind 配置 |
| `.eslintrc.json` | ESLint 规则 |
| `.prettierrc` | 格式化规则 |
| `commitlint.config.js` | Commit 规范 |

---

## 📝 变更日志

| 日期 | 变更 | 变更者 |
|------|------|--------|
| YYYY-MM-DD | 初版 | R3 |
| | | |

---

> 🎯 **核心提醒**: 
> **目录结构是项目的"骨骼"**。骨骼正了，肌肉（代码）自然长得好。
> **新人 30 分钟能理解的结构 = 好结构。**
