# 部署方案 (Deployment Strategy)

> 本文件是 R7 运维迭代官的核心产出之一。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| 部署平台 | `[Vercel / Railway / AWS / 自建]` |
| 最后更新 | `YYYY-MM-DD` |

---

## 一、部署架构

### 环境清单

| 环境 | URL | 用途 | 数据 | 自动部署 |
|------|-----|------|------|---------|
| Local | localhost | 开发 | 本地 | - |
| Preview | pr-xxx.app | PR 预览 | 隔离 | 自动 |
| Staging | staging.app | 预发 | 近生产 | 合并 main |
| Production | app.com | 正式 | 生产 | 手动触发 |

### 基础设施示意图

```
[用户] 
  → [Cloudflare CDN]
  → [Vercel Edge]
  → [Next.js 应用]
  → [Supabase PostgreSQL]
  → [Upstash Redis]
  → [Cloudflare R2]
```

---

## 二、部署流水线

### CI 流程（每次 PR）

```yaml
步骤:
1. Checkout 代码
2. 安装依赖 (pnpm install --frozen-lockfile)
3. Lint (pnpm lint)
4. Type Check (pnpm typecheck)
5. 单元 + 集成测试 (pnpm test)
6. 构建 (pnpm build)
7. E2E 测试 (pnpm test:e2e)
8. 部署 Preview 环境
9. 通知 PR
```

### CD 流程（合并到 main）

```yaml
步骤:
1. 触发 CI 全流程
2. 部署到 Staging
3. 运行 Smoke Test
4. 等待手动批准
5. 部署到 Production
6. 运行生产 Smoke Test
7. 通知团队
```

---

## 三、环境变量管理

### 变量清单（示例）

| 变量 | 用途 | 环境 | 是否敏感 |
|------|------|------|---------|
| `DATABASE_URL` | 数据库 | 全部 | ✅ |
| `JWT_SECRET` | 签名 | 全部 | ✅ |
| `REDIS_URL` | 缓存 | 全部 | ✅ |
| `OAUTH_GOOGLE_CLIENT_ID` | OAuth | 全部 | ❌ |
| `OAUTH_GOOGLE_SECRET` | OAuth | 全部 | ✅ |

### 管理工具
- **Vercel**: Vercel Environment Variables
- **本地**: `.env.local` (不入 Git)
- **模板**: `.env.example` (入 Git)

---

## 四、部署检查清单

### 上线前必查

- [ ] 所有 CI 检查通过
- [ ] 代码审查通过 (R9)
- [ ] 安全扫描无高危 (R6)
- [ ] 数据库迁移已测试
- [ ] 环境变量已配置
- [ ] 回滚方案确认
- [ ] 监控告警已配置
- [ ] 文档已更新
- [ ] 发布日志已准备

### 上线中步骤

1. 备份当前生产数据库
2. 运行数据库迁移（如有）
3. 部署新版本
4. 运行 Smoke Test
5. 逐步放量（如支持 canary）
6. 监控关键指标 30 分钟

### 上线后验证

- [ ] 关键页面可访问
- [ ] 核心功能可用
- [ ] 监控无异常告警
- [ ] 错误率无飙升
- [ ] 性能指标正常

---

## 五、回滚策略

### 触发回滚的条件

- 错误率飙升（> [X]%）
- 关键功能不可用
- 安全漏洞被利用
- 数据完整性问题

### 回滚步骤

**代码回滚**（快）:
1. Vercel: 点击 "Promote" 旧版本
2. 验证恢复

**数据库回滚**（慢，需谨慎）:
1. 评估影响
2. 执行迁移 down 或手动补数据
3. 验证

### 回滚演练

- 频率: 每季度一次
- 流程: 全员参与 + 记录时间

---

## 六、数据库迁移

### 迁移原则

- ✅ 向下兼容（先加字段再用）
- ✅ 可回滚（有 down 迁移）
- ✅ 小步快跑（避免大变更）
- ❌ 不在迁移里跑数据转换（用脚本）

### 迁移流程

```
本地开发
   ↓
PR 审查迁移 SQL
   ↓
CI 中在测试 DB 跑一次
   ↓
合并 main 后 staging 自动执行
   ↓
staging 验证
   ↓
生产执行（人工触发）
   ↓
验证 + 监控
```

### 破坏性变更处理

见 `DATABASE.md` 第六章。

---

## 七、Secrets 管理

### 存储

- **Vercel**: Environment Variables (encrypted at rest)
- **GitHub Actions**: Repository Secrets
- **本地**: `.env.local` (忽略)

### 轮换

| Secret | 轮换周期 | 触发 |
|--------|---------|------|
| JWT_SECRET | 季度 | 日程 |
| DB 密码 | 半年 | 日程 |
| API Keys | 按需 | 疑似泄露 |

### 泄露响应

1. 立即吊销/重置
2. 评估影响范围
3. 通知相关方
4. 根因分析

---

## 八、配置管理

### 运行时配置 vs 构建时配置

**构建时**（不常变）:
- API 端点
- 第三方 SDK 配置

**运行时**（需要动态）:
- Feature flags
- 限流阈值
- 运营配置

### Feature Flags

- **工具**: [内置 / PostHog / LaunchDarkly]
- **用途**: 灰度发布、A/B 测试、快速关闭
- **规范**: 功能发布后 2 周内移除旧代码

---

## 九、域名与 SSL

- **主域名**: [...]
- **DNS 服务商**: [...]
- **SSL 证书**: Cloudflare / Let's Encrypt（自动续期）
- **重定向规则**: www → 根域名 / http → https

---

## 十、备份与灾备

### 数据备份

- 自动快照: 每日
- 保留: 30 天
- 异地备份: [如有]
- 恢复演练: 季度

### 灾难恢复 (DR)

- **RTO**: < 1 小时
- **RPO**: < 15 分钟
- **预案**: [详见 DR 手册]

---

## 📖 相关文档

- [`MONITORING.md`](./MONITORING.md)
- [`../04-architecture/ARCHITECTURE.md`](../04-architecture/ARCHITECTURE.md)

---

> 🎯 **核心提醒**: 
> **部署不是"一次性事件"，是"持续性能力"**。
> **能 10 分钟内回滚的部署，才是安全的部署。**
