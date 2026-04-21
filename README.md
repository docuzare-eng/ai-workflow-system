# AI-Workflow-System

> 一套系统化的 AI 驱动项目开发工作流，让复杂项目从想法到落地全程可控。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Active Development](https://img.shields.io/badge/Status-Active%20Development-green.svg)]()

---

## 🎯 这是什么？

**AI-Workflow-System** 是一套用于指挥 AI（Claude、Claude Code 等）开发复杂项目的**标准化工作流系统**。

它解决了 AI 辅助开发中的核心痛点：
- ❌ 上下文丢失、AI "失忆"
- ❌ 缺乏系统性规划，代码越写越乱
- ❌ 大项目中途失控、推倒重来
- ❌ 决策无据可依、质量失控

通过 **7 阶段工作流 + 12 个专业角色 + 分层记忆系统 + 质量门禁**，让 AI 真正成为你的"工程团队"，而不是"随叫随到的打字员"。

---

## 💡 核心理念

> **AI 负责执行，你负责判断。流程是思考的外化，文档是 AI 的记忆。**

- **文档驱动开发**：先规划、后实现，文档即上下文
- **角色专业化**：每个阶段都有专属的 AI 角色和指令模板
- **小步快跑**：每一步都可验证、可回滚
- **双向回流**：允许后置阶段反馈前置阶段问题
- **人机协作**：你是总指挥，AI 是专业团队

---

## 🏗️ 系统架构┌─────────────────────────────────────────────────────────┐
│  协调层：R8 项目协调官（你的 AI CTO）                    │
│  贯穿角色：R9 代码审查员 / R10 文档守护者 / R11 风险侦察 │
└─────────────────────────────────────────────────────────┘
↓              ↓              ↓
┌──────────────────────────────────────────────────────────┐
│  工作流 7 阶段                                            │
│                                                           │
│  Stage 0: 可行性验证    → R0 可行性分析师                 │
│  Stage 1: 构想打磨      → R1 产品构想师                   │
│  Stage 2: 需求规格化    → R2 需求分析师                   │
│  Stage 3: 架构设计      → R3 架构师                       │
│  Stage 4: 骨架搭建      → R4 工程骨架师                   │
│  Stage 5: 功能开发      → R5 开发工程师（循环）           │
│  Stage 6: 质量保障      → R6 QA 工程师                    │
│  Stage 7: 迭代演进      → R7 运维迭代师                   │
└──────────────────────────────────────────────────────────┘
↓              ↓              ↓
┌──────────────────────────────────────────────────────────┐
│  共享记忆层 (.ai-workflow/00-core/)                       │
│  PROJECT_CONSTITUTION / DECISIONS / CONSTRAINTS / GLOSSARY│
└──────────────────────────────────────────────────────────┘

---

## 📂 仓库结构ai-workflow-system/
├── README.md                    # 你正在看的文件
├── CHANGELOG.md                 # 版本变更记录
├── LICENSE                      # MIT 协议
│
├── docs/                        # 📚 系统文档
│   ├── PHILOSOPHY.md           # 设计哲学
│   ├── WORKFLOW_OVERVIEW.md    # 工作流全景图
│   ├── ROLES.md                # 角色体系说明
│   ├── GETTING_STARTED.md      # 快速上手
│   ├── BEST_PRACTICES.md       # 最佳实践
│   └── TROUBLESHOOTING.md      # 常见问题
│
├── template/                    # 🎁 项目模板（开新项目时复制这个）
│   └── .ai-workflow/           # 工作流元数据目录
│       ├── 00-core/            # 共享记忆层
│       ├── 01-feasibility/     # 阶段产出
│       ├── ... (02~08)
│       ├── meta/               # 工作流状态管理
│       └── logs/               # 决策与变更日志
│
├── prompts/                     # 🤖 角色指令模板
│   ├── stages/                 # 8 个阶段角色 Prompt
│   ├── cross-cutting/          # 4 个贯穿角色 Prompt
│   └── shared/                 # 共享指令片段
│
├── quality-gates/               # ✅ 阶段门禁清单
│
└── examples/                    # 💼 示例项目（后续补充）

---

## 🚀 快速开始

> ⚠️ 本系统目前处于活跃开发阶段，使用指南将在系统搭建完成后完善。

### 开新项目的流程（未来使用方式）

1. **复制模板**：把 `template/` 目录复制到你的新项目中
2. **启动协调官**：开一个 Claude.ai 对话窗口，粘贴 `prompts/cross-cutting/R8-project-coordinator.md`
3. **按阶段推进**：依次启动 R0 → R7 各阶段角色
4. **通过门禁**：每个阶段结束前用 `quality-gates/` 中的清单自检
5. **持续迭代**：保持 `00-core/` 中的核心档案常新

详细步骤参见 [`docs/GETTING_STARTED.md`](./docs/GETTING_STARTED.md)（开发中）

---

## 📖 核心文档索引

| 文档 | 内容 | 优先级 |
|------|------|--------|
| [PHILOSOPHY.md](./docs/PHILOSOPHY.md) | 为什么这样设计？核心理念 | ⭐⭐⭐⭐⭐ |
| [WORKFLOW_OVERVIEW.md](./docs/WORKFLOW_OVERVIEW.md) | 7 阶段工作流详解 | ⭐⭐⭐⭐⭐ |
| [ROLES.md](./docs/ROLES.md) | 12 个角色的职责分工 | ⭐⭐⭐⭐⭐ |
| [GETTING_STARTED.md](./docs/GETTING_STARTED.md) | 新手上路指南 | ⭐⭐⭐⭐ |
| [BEST_PRACTICES.md](./docs/BEST_PRACTICES.md) | 最佳实践汇总 | ⭐⭐⭐ |
| [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) | 常见问题排查 | ⭐⭐⭐ |

---

## 🛤️ 路线图

- [x] Stage 0：搭建项目骨架
- [ ] Stage 1：编写顶层文档
- [ ] Stage 2：编写核心档案模板（00-core）
- [ ] Stage 3：编写 8 个阶段角色 Prompt
- [ ] Stage 4：编写 4 个贯穿角色 Prompt
- [ ] Stage 5：编写产出物模板 + 门禁清单
- [ ] Stage 6：编写使用指南与示例
- [ ] Stage 7：实战验证（用真实项目跑一遍）
- [ ] Stage 8：开源社区版本（可选）

查看 [CHANGELOG.md](./CHANGELOG.md) 了解详细进度。

---

## 🤝 贡献

这是一个演进中的系统，欢迎：
- 💡 提 Issue 分享你的想法
- 🐛 报告 Bug 或文档不清之处
- 🔧 提 PR 优化流程或 Prompt

---

## 📜 许可证

[MIT License](./LICENSE) - 自由使用、修改、分发。

---

## 🌟 致谢

本系统的设计灵感来自：
- **软件工程经典方法论**：Agile、DDD、ADR
- **LLM 协作最佳实践**：Prompt Engineering、Context Engineering
- **真实项目踩过的坑**：无数次推倒重来换来的教训

> "AI 不是替代思考，而是放大思考。"