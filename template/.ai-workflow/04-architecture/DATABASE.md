# 数据库设计 (Database Design)

> 本文件定义项目的数据库结构、关系、索引和约束。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| 数据库类型 | `[PostgreSQL 15 / MySQL 8 / ...]` |
| 文档版本 | `v1.0` |
| 最后更新 | `YYYY-MM-DD` |
| Schema 管理 | `[Prisma / Flyway / 原生 SQL]` |

---

## 一、设计原则

### 通用原则

1. **主键策略**: 使用 `cuid` / `uuid` / `自增 ID`（统一选一种）
2. **时间戳**: 所有表必须有 `created_at` 和 `updated_at`
3. **软删除**: 涉及业务重要数据使用 `deleted_at` 软删除
4. **审计**: 涉及用户行为的表添加 `created_by`, `updated_by`
5. **命名规范**:
   - 表名: `snake_case`，复数形式（`users`, `posts`）
   - 字段名: `snake_case`（`user_id`, `created_at`）
   - 索引名: `idx_[表]_[字段]`

### 性能原则

- 高频查询必须走索引
- 大表（> 1000 万行）考虑分区
- 避免 SELECT *（显式列出字段）
- 避免 N+1 查询（用 JOIN 或 batch）

### 安全原则

- 敏感字段加密存储（密码、令牌、身份证号）
- 不存储明文密码（使用 bcrypt / argon2）
- 生产数据脱敏用于测试

---

## 二、实体关系图 (ERD)

```mermaid
erDiagram
    USER ||--o{ POST : creates
    USER ||--o{ COMMENT : writes
    POST ||--o{ COMMENT : has
    USER {
        string id PK
        string email UK
        string password_hash
        string name
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    POST {
        string id PK
        string user_id FK
        string title
        text content
        string status
        timestamp created_at
        timestamp updated_at
    }
    COMMENT {
        string id PK
        string post_id FK
        string user_id FK
        text content
        timestamp created_at
    }
```

> 💡 根据实际项目替换为真实的实体关系。

---

## 三、核心表定义

### 3.1 users (用户表)

**用途**: 存储用户基本信息

**字段**:

| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| id | VARCHAR(25) | PRIMARY KEY | cuid() | 主键 |
| email | VARCHAR(255) | UNIQUE, NOT NULL | - | 邮箱（登录用） |
| password_hash | VARCHAR(255) | NOT NULL | - | 密码 bcrypt 哈希 |
| name | VARCHAR(100) | NOT NULL | - | 用户名 |
| avatar_url | VARCHAR(500) | NULL | - | 头像 URL |
| email_verified_at | TIMESTAMP | NULL | - | 邮箱验证时间 |
| role | VARCHAR(20) | NOT NULL | 'user' | 角色 (user/admin) |
| status | VARCHAR(20) | NOT NULL | 'active' | 状态 (active/banned) |
| created_at | TIMESTAMP | NOT NULL | NOW() | 创建时间 |
| updated_at | TIMESTAMP | NOT NULL | NOW() | 更新时间 |
| deleted_at | TIMESTAMP | NULL | - | 软删除时间 |

**索引**:
- `PRIMARY KEY (id)`
- `UNIQUE INDEX idx_users_email (email)` ← 登录查询
- `INDEX idx_users_created_at (created_at)` ← 按时间排序
- `INDEX idx_users_deleted_at (deleted_at)` ← 软删除过滤

**约束**:
- email 格式必须合法（应用层校验）
- password_hash 长度固定（bcrypt 60 字符）
- role 必须是 enum 值

**业务规则**:
- 删除账号使用软删除（deleted_at），不物理删除
- 邮箱修改需重新验证

**数据量级预估**:
- MVP: [X] 条
- 1 年后: [X] 条
- 增长率: [...]

---

### 3.2 [表名]

**用途**: [...]

**字段**:
| 字段 | 类型 | 约束 | 默认值 | 说明 |
|------|------|------|--------|------|
| ... | ... | ... | ... | ... |

**索引**:
- [...]

**约束**:
- [...]

**业务规则**:
- [...]

---

### 3.3 [表名]

[同上]

---

## 四、关系设计

### 一对多 (One-to-Many)

**示例**: 用户-文章
```
User.id (1) ←———— (N) Post.user_id
```

**实现**:
- `posts.user_id` 引用 `users.id`
- 外键约束: `ON DELETE CASCADE` 或 `ON DELETE SET NULL`（根据业务）

### 多对多 (Many-to-Many)

**示例**: 文章-标签（需中间表）
```
Post.id (1) ←— (N) PostTag (N) —→ (1) Tag.id
```

**中间表**:
```
post_tags (
  post_id FK REFERENCES posts(id),
  tag_id FK REFERENCES tags(id),
  created_at TIMESTAMP,
  PRIMARY KEY (post_id, tag_id)
)
```

---

## 五、索引策略

### 5.1 索引规划原则

- **主键**: 自动索引
- **外键**: 必须索引
- **UNIQUE 约束**: 自动索引
- **高频查询字段**: 显式索引
- **排序字段**: 考虑索引（特别是分页）
- **过滤字段**: 选择性高的优先

### 5.2 不要滥用索引

- ❌ 不给所有字段加索引（写入性能下降）
- ❌ 低选择性字段（如性别、布尔）单独索引意义不大
- ❌ 频繁更新的字段避免索引

### 5.3 复合索引

**规则**: 把选择性高的字段放前面

**示例**:
```sql
-- 查询: WHERE user_id = ? AND status = ? ORDER BY created_at DESC
CREATE INDEX idx_posts_user_status_created 
ON posts (user_id, status, created_at DESC);
```

---

## 六、迁移策略

### 6.1 迁移工具

使用 [Prisma Migrate / Flyway / 原生 SQL] 管理 schema 变更。

### 6.2 迁移流程

```
1. 开发: 修改 schema 文件
         ↓
2. 生成: pnpm prisma migrate dev  (本地测试)
         ↓
3. 审查: 人工审查生成的 SQL
         ↓
4. 测试: 在 staging 环境执行
         ↓
5. 生产: 生产环境执行
         ↓
6. 验证: 数据完整性 + 功能验证
```

### 6.3 迁移最佳实践

- ✅ **向下兼容**: 先加字段再用，先不用再删
- ✅ **可回滚**: 每个迁移必须可逆
- ✅ **小步快跑**: 大变更拆成多个小迁移
- ✅ **生产备份**: 执行前必须备份
- ❌ **避免重命名**: 先加新字段，迁移数据，再删旧字段
- ❌ **避免数据丢失操作**: DROP COLUMN 前确认无引用

### 6.4 破坏性变更处理

| 操作 | 策略 |
|------|------|
| 删除字段 | 分 2 个版本：先标记废弃、后删除 |
| 重命名字段 | 添加新字段 + 数据迁移 + 删除旧字段 |
| 修改字段类型 | 新增字段 + 迁移 + 切换 + 删除旧 |
| 重构表 | 双写 + 数据迁移 + 切换读取 + 删除旧表 |

---

## 七、数据库连接与池

### 7.1 连接配置

```typescript
// 示例配置
DATABASE_URL=postgresql://user:pass@host:5432/db
DATABASE_POOL_MAX=10       // 最大连接数
DATABASE_POOL_MIN=2        // 最小连接数
DATABASE_POOL_IDLE=30000   // 空闲超时(ms)
DATABASE_TIMEOUT=5000      // 查询超时(ms)
```

### 7.2 连接池调优

- 连接数 ≈ (核数 × 2) + 有效磁盘轴数
- 对于 Serverless: 使用连接池代理 (PgBouncer)

---

## 八、备份与恢复

### 8.1 备份策略

| 类型 | 频率 | 保留 | 位置 |
|------|------|------|------|
| 自动快照 | 每日 | 30 天 | 托管服务 |
| 全量备份 | 每周 | 3 个月 | 冷存储 |
| 重大变更前 | 手动 | 1 个月 | 冷存储 |

### 8.2 恢复演练

- **频率**: 每季度一次
- **目标**: 验证备份可用 + 团队熟悉流程
- **RTO** (恢复时间目标): < 1 小时
- **RPO** (恢复点目标): < 15 分钟

---

## 九、种子数据 (Seed Data)

### 9.1 开发环境种子

- 演示用户: `demo@example.com / demo123`
- 示例数据: [N] 条记录，覆盖所有业务场景

### 9.2 测试环境种子

- 自动化测试通过 factory 生成
- 清理规则: 每个测试用例独立数据

### 9.3 生产环境种子

- 初始管理员账号
- 基础配置数据

---

## 十、性能监控

### 10.1 需要监控的指标

- 查询 P95 延迟
- 慢查询数量（> 1s）
- 连接池使用率
- 死锁次数
- 数据库大小增长

### 10.2 慢查询处理流程

```
监控发现慢查询
    ↓
EXPLAIN ANALYZE
    ↓
是否缺索引？→ 加索引
    ↓
是否 N+1？→ 优化查询
    ↓
是否数据量过大？→ 分页/归档
    ↓
是否设计问题？→ 重新设计
```

---

## 📝 变更日志

| 日期 | 变更 | 迁移编号 |
|------|------|---------|
| YYYY-MM-DD | 初版 | - |
| YYYY-MM-DD | 新增 xxx 表 | 20260421_xxx |
| | | |

---

## 📖 相关文档

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) - 整体架构
- [`TECH_STACK.md`](./TECH_STACK.md) - ORM 选型
- [`SECURITY_DESIGN.md`](./SECURITY_DESIGN.md) - 数据安全

---

> 🎯 **核心提醒**: 
> **数据库设计改起来最痛苦**——数据迁移成本极高。
> **前期多花 2 小时设计，后期省 200 小时重构。**
