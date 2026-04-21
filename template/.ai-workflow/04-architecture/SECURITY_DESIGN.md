# 安全设计 (Security Design)

> 本文件定义项目的安全威胁模型和防御设计。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| 文档版本 | `v1.0` |
| 最后更新 | `YYYY-MM-DD` |
| 安全审查 | `[初设 / 已审 / 已测]` |

---

## 一、安全理念

### 纵深防御 (Defense in Depth)

不依赖单一防御手段，用多层防御叠加：

```
┌────────────────────────────────────┐
│ 1. 边缘层 (WAF, DDoS)             │
├────────────────────────────────────┤
│ 2. 网络层 (HTTPS, Firewall)       │
├────────────────────────────────────┤
│ 3. 应用层 (认证、授权、验证)        │
├────────────────────────────────────┤
│ 4. 数据层 (加密、访问控制)          │
├────────────────────────────────────┤
│ 5. 监控层 (异常检测、审计)          │
└────────────────────────────────────┘
```

### 最小权限原则

- 用户/服务只获得完成工作所需的最少权限
- 默认拒绝，显式允许

### 零信任

- 不信任任何输入
- 不信任任何请求（即使内网）
- 每次操作都验证

---

## 二、威胁建模 (STRIDE)

### 2.1 核心资产

| 资产 | 重要性 | 保护重点 |
|------|-------|---------|
| 用户账号 | 极高 | 不被盗用 |
| 用户数据 | 高 | 隐私、完整性 |
| 业务数据 | 高 | 防篡改、可用性 |
| 密钥/Token | 极高 | 不泄露 |
| 服务可用性 | 高 | 不宕机 |

### 2.2 STRIDE 威胁分析

#### 🎭 Spoofing (身份伪造)

**场景**: 攻击者冒充其他用户

**可能手段**:
- 盗取密码
- 窃取 Token
- CSRF 攻击
- Session 劫持

**防御**:
- ✅ 强密码要求（bcrypt 哈希）
- ✅ HTTPS 强制
- ✅ HttpOnly + Secure + SameSite Cookie
- ✅ CSRF Token
- ✅ 2FA（后续引入）
- ✅ 异常登录检测

#### ⚒️ Tampering (篡改)

**场景**: 修改传输或存储的数据

**可能手段**:
- 中间人攻击
- 参数篡改
- 数据库注入

**防御**:
- ✅ HTTPS 全站
- ✅ 参数化查询（防 SQL 注入）
- ✅ 输入验证（Zod schema）
- ✅ 关键字段签名
- ✅ 审计日志

#### 📜 Repudiation (抵赖)

**场景**: 用户否认做过操作

**防御**:
- ✅ 详细的审计日志
- ✅ 操作时间戳
- ✅ 关键操作的不可篡改记录

#### 💧 Information Disclosure (信息泄露)

**场景**: 看到不该看到的数据

**可能手段**:
- 越权访问（垂直/水平）
- 错误信息泄露堆栈
- 未加密的敏感数据
- 日志中的敏感信息

**防御**:
- ✅ 严格的权限检查
- ✅ 错误信息脱敏
- ✅ 敏感字段加密存储
- ✅ 日志脱敏
- ✅ 数据库加密连接

#### 🚫 Denial of Service (拒绝服务)

**场景**: 使服务不可用

**可能手段**:
- DDoS 攻击
- 资源耗尽攻击
- 慢请求攻击

**防御**:
- ✅ Rate limiting
- ✅ CDN + DDoS 防护
- ✅ 超时控制
- ✅ 资源配额
- ✅ 优雅降级

#### 🔑 Elevation of Privilege (权限提升)

**场景**: 普通用户获得管理员权限

**可能手段**:
- 权限检查漏洞
- 默认权限错误
- 内部接口暴露

**防御**:
- ✅ 严格的 RBAC
- ✅ 每个接口显式权限检查
- ✅ 权限变更审计
- ✅ 最小权限原则

---

## 三、具体防御措施

### 3.1 认证 (Authentication)

#### 密码安全

- **存储**: bcrypt（cost ≥ 12）或 argon2id
- **强度要求**:
  - 最少 8 位
  - 包含字母 + 数字
  - 不允许常见弱密码（字典检查）
- **限制**:
  - 登录失败 5 次锁定 15 分钟
  - 密码修改需当前密码验证

#### Session/Token 管理

**Token 策略**:
- Access Token: JWT, 1 小时有效
- Refresh Token: 30 天有效，单次使用
- Token 撤销: 黑名单 + Redis

**Cookie 配置**:
```
HttpOnly: true        # 防 JS 读取
Secure: true          # 仅 HTTPS
SameSite: Lax/Strict  # 防 CSRF
Path: /
MaxAge: [合适值]
```

#### 多因素认证 (2FA)

- **MVP**: 可选实现
- **未来**: TOTP（Google Authenticator 等）
- **关键操作**: 敏感操作强制 2FA

### 3.2 授权 (Authorization)

#### 权限模型

**选择**: `[RBAC / ABAC / 简单角色]`

**角色定义**（RBAC 示例）:
- `guest`: 访客
- `user`: 普通用户
- `premium`: 付费用户
- `admin`: 管理员

#### 权限检查

**多层检查**:
1. **中间件层**: 认证检查（是否登录）
2. **路由层**: 角色检查（是否有权访问）
3. **Service 层**: 资源检查（是否自己的资源）

```typescript
// 示例：三层检查
async function deletePost(postId: string, userId: string) {
  // Service 层：资源所有权检查
  const post = await postRepo.findById(postId);
  if (post.userId !== userId) {
    throw new ForbiddenError('不能删除他人文章');
  }
  await postRepo.delete(postId);
}
```

### 3.3 输入验证与输出编码

#### 输入验证

- **统一入口**: 所有 API 入口通过 Zod schema 验证
- **白名单**: 定义合法值范围，拒绝其他
- **类型检查**: TypeScript + 运行时验证双保险

#### SQL 注入防御

- ✅ 使用 ORM（Prisma）或参数化查询
- ❌ 禁止字符串拼接 SQL

```typescript
// ❌ 危险
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ 安全
db.query('SELECT * FROM users WHERE email = ?', [email]);
```

#### XSS 防御

- **React 默认转义**: 默认安全
- **危险源**: `dangerouslySetInnerHTML`、`innerHTML`
- **白名单**: 富文本用 DOMPurify 清洗
- **CSP**: Content Security Policy 头部

#### CSRF 防御

- **SameSite Cookie**: 主要防御
- **CSRF Token**: 额外防御（表单提交）
- **Origin/Referer 检查**: 补充

### 3.4 数据保护

#### 传输加密

- ✅ HTTPS 全站（HSTS 强制）
- ✅ TLS 1.2+ 最低版本
- ✅ 内部服务间通信也加密

#### 存储加密

**必须加密的字段**:
- 密码（bcrypt）
- API Token
- 用户敏感信息（身份证、电话）
- 支付信息

**加密方式**:
- 对称加密: AES-256-GCM
- 密钥管理: 环境变量 / KMS / Vault

#### 数据脱敏

**日志中脱敏**:
```typescript
// ❌ 日志中敏感信息
logger.info('用户登录', { email, password });

// ✅ 脱敏
logger.info('用户登录', { email: 'a***@example.com' });
```

**响应脱敏**:
- 不返回 password_hash
- 不返回内部 ID
- 不返回详细的错误堆栈（生产环境）

### 3.5 访问控制

#### Rate Limiting

| 范围 | 限制 | 目的 |
|------|------|------|
| 全局 IP | 1000 req/min | 防 DDoS |
| 认证 API | 60 req/min per IP | 防爆破 |
| 登录 | 5 次/15 分钟 per IP | 防爆破 |
| 敏感操作 | 10 次/小时 per 用户 | 防滥用 |

#### IP 限制（如需要）

- 管理后台: IP 白名单
- 敏感 API: 地域限制

### 3.6 密钥管理

#### 密钥类型

- **JWT Secret**: 签名 JWT
- **数据库密码**: 连接 DB
- **第三方 API Key**: OAuth、邮件、支付
- **加密密钥**: AES 加密

#### 密钥存储

- ✅ 环境变量 (.env, 不入 Git)
- ✅ 密钥管理服务 (KMS, Vault)
- ❌ **绝对不能**硬编码
- ❌ **绝对不能**入 Git

#### 密钥轮换

| 密钥 | 轮换频率 |
|------|---------|
| JWT Secret | 每季度 |
| 数据库密码 | 每半年 |
| API Key | 按需 |

### 3.7 安全头部 (Security Headers)

必须配置的 HTTP 头：

```
Strict-Transport-Security: max-age=63072000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: [根据项目配置]
Permissions-Policy: [按需限制 API]
```

### 3.8 依赖安全

#### 供应链安全

- ✅ 锁定版本（pnpm-lock.yaml 提交）
- ✅ 定期 `npm audit`（CI 自动）
- ✅ 订阅 GitHub Security Advisories
- ✅ 使用 Dependabot / Renovate 自动 PR

#### 新增依赖审查

- 包的受欢迎度（下载量）
- 维护活跃度（更新频率）
- 许可证兼容性
- 已知漏洞

---

## 四、安全开发实践

### 4.1 代码审查中的安全检查

R9 代码审查员必须检查：
- [ ] SQL 是否参数化？
- [ ] 用户输入是否验证？
- [ ] 权限检查是否到位？
- [ ] 错误信息是否脱敏？
- [ ] 日志是否脱敏？
- [ ] 敏感操作是否有审计？

### 4.2 CI 中的安全检查

- [ ] 依赖漏洞扫描 (npm audit)
- [ ] 密钥泄露扫描 (gitleaks)
- [ ] SAST (静态代码分析)
- [ ] License 合规检查

### 4.3 部署安全

- ✅ 不同环境不同密钥
- ✅ 生产环境最小权限
- ✅ 数据库不暴露公网
- ✅ 管理后台独立域名 + IP 白名单
- ✅ 定期渗透测试（重要项目）

---

## 五、事件响应

### 5.1 安全事件分级

| 级别 | 定义 | 响应时间 |
|------|------|---------|
| P0 | 数据泄露、服务瘫痪 | 立即 |
| P1 | 高危漏洞被利用 | < 1 小时 |
| P2 | 发现高危漏洞 | < 24 小时 |
| P3 | 发现中低危漏洞 | < 1 周 |

### 5.2 事件响应流程

```
发现事件
  ↓
立即止血（隔离/禁用）
  ↓
评估影响范围
  ↓
通知相关方（用户/法律）
  ↓
根因分析
  ↓
永久修复
  ↓
事后复盘 (Postmortem)
```

### 5.3 事件响应清单

**数据泄露响应**:
- [ ] 立即隔离受影响系统
- [ ] 评估泄露数据范围
- [ ] 通知受影响用户（GDPR 72 小时）
- [ ] 报告监管机构（如需要）
- [ ] 修复漏洞
- [ ] 全面审计

---

## 六、合规要求

### 6.1 地域法规

根据项目目标地区：
- **欧盟**: GDPR
- **加州**: CCPA/CPRA
- **中国**: 个人信息保护法、网络安全法
- **全球**: 行业特定（HIPAA, PCI-DSS 等）

### 6.2 合规清单

- [ ] 隐私政策明确
- [ ] 用户同意收集
- [ ] 数据导出能力
- [ ] 数据删除能力
- [ ] 数据出境合规
- [ ] 儿童保护（如涉及）

---

## 七、安全审计日志

### 7.1 必须记录的事件

- 登录成功/失败
- 密码修改
- 权限变更
- 敏感操作（支付、删除）
- 管理员操作
- 异常行为

### 7.2 日志格式

```json
{
  "timestamp": "2026-04-21T10:30:00Z",
  "event": "user.login",
  "userId": "cuid123",
  "ip": "1.2.3.4",
  "userAgent": "Mozilla/...",
  "result": "success",
  "metadata": {}
}
```

### 7.3 日志保留

- 一般日志: 30 天
- 安全日志: 1 年
- 审计日志: 3 年（或法规要求）

---

## 八、持续安全

### 8.1 安全检查节奏

- **每日**: 自动化依赖扫描
- **每周**: 日志审查（异常模式）
- **每月**: 权限审查
- **每季度**: R11 风险侦察兵全面扫描
- **每年**: 渗透测试（重要项目）

### 8.2 安全培训

- 开发前阅读本文档
- OWASP Top 10 了解
- 定期 Security Awareness 回顾

---

## 📝 变更日志

| 日期 | 变更 | 变更者 |
|------|------|--------|
| YYYY-MM-DD | 初版 | R3 |
| | | |

---

## 📖 相关文档

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) - 整体架构
- [`API_DESIGN.md`](./API_DESIGN.md) - API 安全
- [`../07-quality/SECURITY_AUDIT.md`](../07-quality/SECURITY_AUDIT.md) - 安全审计
- [`../00-core/CONSTRAINTS.md`](../00-core/CONSTRAINTS.md) - 安全约束

---

> 🎯 **核心提醒**: 
> **安全不是一次性的事，是持续的实践。**
> **一个漏洞的成本 >> 提前防御的成本。**
> **记住：你不需要"无敌"，但要比攻击者投入的成本更高。**
