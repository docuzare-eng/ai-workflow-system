# API 设计 (API Design)

> 本文件定义项目的 API 规范、端点清单、错误处理和认证授权。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| API 风格 | `[REST / GraphQL / tRPC]` |
| 版本策略 | `URL 前缀 /api/v1/` |
| 文档版本 | `v1.0` |
| 最后更新 | `YYYY-MM-DD` |

---

## 一、API 风格选择

### 采用: `[REST / GraphQL / tRPC]`

**选择理由** (详见 DECISIONS.md ADR-XXX):
1. [理由 1]
2. [理由 2]

**考虑过但未选**:
- [其他方案及放弃理由]

---

## 二、通用规范

### 2.1 URL 规范

#### 路径风格
- 使用**资源名词**（复数）: `/users` ✅ 而非 `/getUser` ❌
- 使用**小写字母** + **连字符**: `/user-profiles` ✅
- 嵌套层级不超过 2 层: `/users/:id/posts` ✅ 而非 `/users/:id/posts/:pid/comments` ❌

#### 版本管理
- 所有端点必须带版本: `/api/v1/...`
- 大版本升级保留旧版本至少 6 个月

#### 示例路径

```
GET    /api/v1/users              # 获取用户列表
GET    /api/v1/users/:id          # 获取某用户
POST   /api/v1/users              # 创建用户
PATCH  /api/v1/users/:id          # 部分更新
PUT    /api/v1/users/:id          # 完整替换
DELETE /api/v1/users/:id          # 删除

GET    /api/v1/users/:id/posts    # 用户的文章列表
```

### 2.2 HTTP 方法规范

| 方法 | 用途 | 幂等 | 安全 |
|------|------|------|------|
| GET | 读取 | ✅ | ✅ |
| POST | 创建 | ❌ | ❌ |
| PUT | 完整替换 | ✅ | ❌ |
| PATCH | 部分更新 | ❌ | ❌ |
| DELETE | 删除 | ✅ | ❌ |

### 2.3 状态码规范

#### 成功 (2xx)
- `200 OK`: 请求成功（GET, PATCH, PUT）
- `201 Created`: 资源已创建（POST）
- `204 No Content`: 成功但无返回体（DELETE）

#### 客户端错误 (4xx)
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 未认证（未登录）
- `403 Forbidden`: 已认证但无权限
- `404 Not Found`: 资源不存在
- `409 Conflict`: 冲突（如重复注册）
- `422 Unprocessable Entity`: 验证失败（字段格式错误）
- `429 Too Many Requests`: 频率限制

#### 服务端错误 (5xx)
- `500 Internal Server Error`: 服务器内部错误
- `502 Bad Gateway`: 上游服务错误
- `503 Service Unavailable`: 服务暂时不可用
- `504 Gateway Timeout`: 上游超时

### 2.4 请求格式

#### 请求头
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>       # 认证
X-Request-ID: <uuid>                # 可选，前端追踪
Accept-Language: zh-CN              # 国际化
```

#### 请求体（POST/PATCH/PUT）
```json
{
  "field1": "value1",
  "field2": 123
}
```

### 2.5 响应格式

#### 成功响应

**单个资源**:
```json
{
  "data": {
    "id": "cuid123",
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

**列表资源**（带分页）:
```json
{
  "data": [
    { "id": "1", "name": "Alice" },
    { "id": "2", "name": "Bob" }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

#### 错误响应（统一格式）

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "用户不存在",
    "details": {
      "userId": "cuid123"
    },
    "requestId": "req_xxx"
  }
}
```

---

## 三、认证与授权

### 3.1 认证方式

**方式**: `[JWT / Session / OAuth 2.0]`

**详情**:
- Token 格式: [JWT / Opaque Token]
- 携带方式: `Authorization: Bearer <token>`
- 有效期: 
  - Access Token: [1 小时]
  - Refresh Token: [30 天]
- 刷新机制: [...]

### 3.2 公开端点 (无需认证)

- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/forgot-password`
- `GET /api/v1/public/...`

### 3.3 受保护端点 (需认证)

所有其他端点默认需要认证。

### 3.4 权限模型

**采用**: `[RBAC / ABAC / 简单角色]`

**角色定义**:
- `guest`: 访客（未登录）
- `user`: 普通用户
- `admin`: 管理员

**权限检查位置**:
- 中间件层: 认证检查
- Service 层: 业务权限检查（所有权等）

### 3.5 Rate Limiting

| 端点类型 | 限制 | 依据 |
|---------|------|------|
| 公开 API | 60 req/min | IP |
| 认证 API | 1000 req/min | 用户 ID |
| 敏感操作（如登录） | 5 req/min | IP |

---

## 四、端点清单

### 🔐 认证模块 (/api/v1/auth)

#### POST /api/v1/auth/register
**用途**: 注册新用户  
**认证**: 无需  
**关联故事**: US-001

**请求**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "name": "Alice"
}
```

**响应 201**:
```json
{
  "data": {
    "user": {
      "id": "cuid123",
      "email": "user@example.com",
      "name": "Alice"
    },
    "tokens": {
      "accessToken": "eyJ...",
      "refreshToken": "eyJ..."
    }
  }
}
```

**错误响应**:
- `400 INVALID_INPUT`: 输入格式错误
- `409 EMAIL_EXISTS`: 邮箱已被注册

---

#### POST /api/v1/auth/login
**用途**: 用户登录  
**认证**: 无需  
**关联故事**: US-002

**请求**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**响应 200**:
```json
{
  "data": {
    "user": { "id": "...", "email": "...", "name": "..." },
    "tokens": {
      "accessToken": "...",
      "refreshToken": "..."
    }
  }
}
```

**错误响应**:
- `401 INVALID_CREDENTIALS`: 邮箱或密码错误
- `429 TOO_MANY_ATTEMPTS`: 尝试次数超限

---

#### POST /api/v1/auth/refresh
**用途**: 刷新 Access Token

[同上格式]

---

#### POST /api/v1/auth/logout
**用途**: 登出

[同上格式]

---

### 👤 用户模块 (/api/v1/users)

#### GET /api/v1/users/me
**用途**: 获取当前用户信息  
**认证**: 需要

[同上格式]

---

#### PATCH /api/v1/users/me
**用途**: 更新当前用户信息

[同上格式]

---

### 📦 [业务模块] (/api/v1/[module])

[列出所有业务相关端点]

---

## 五、错误码表

> 统一的错误码便于前端识别和用户体验优化。

### 5.1 错误码规范

格式: `<MODULE>_<ACTION>_<RESULT>`

示例:
- `AUTH_LOGIN_FAILED`
- `USER_NOT_FOUND`
- `POST_CREATE_LIMIT_EXCEEDED`

### 5.2 通用错误码

| Code | HTTP | 说明 |
|------|------|------|
| `INVALID_INPUT` | 400 | 输入格式错误 |
| `UNAUTHORIZED` | 401 | 未认证 |
| `FORBIDDEN` | 403 | 无权限 |
| `NOT_FOUND` | 404 | 资源不存在 |
| `RATE_LIMIT_EXCEEDED` | 429 | 超频率限制 |
| `INTERNAL_ERROR` | 500 | 服务器错误 |

### 5.3 业务错误码

| Code | HTTP | 说明 |
|------|------|------|
| `EMAIL_EXISTS` | 409 | 邮箱已注册 |
| `INVALID_CREDENTIALS` | 401 | 凭证无效 |
| `EMAIL_NOT_VERIFIED` | 403 | 邮箱未验证 |
| `QUOTA_EXCEEDED` | 403 | 配额超限 |
| `[其他]` | - | [...] |

---

## 六、分页规范

### 6.1 游标分页（推荐用于时间序列）

**请求**:
```
GET /api/v1/posts?cursor=cuid123&limit=20
```

**响应**:
```json
{
  "data": [...],
  "pagination": {
    "nextCursor": "cuid456",
    "hasMore": true
  }
}
```

### 6.2 偏移分页（用于页码跳转）

**请求**:
```
GET /api/v1/posts?page=2&pageSize=20
```

**响应**:
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

## 七、输入验证

### 7.1 验证原则

- **客户端验证**: 友好提示（可被绕过）
- **服务端验证**: 安全保障（不可绕过）
- **前后双重**: 是最佳实践

### 7.2 验证工具

使用 **Zod** 定义 schema，前后端共享：

```typescript
const UserRegisterSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(100)
});
```

### 7.3 验证错误响应

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "输入验证失败",
    "details": {
      "fields": [
        { "field": "email", "message": "邮箱格式不正确" },
        { "field": "password", "message": "密码至少 8 位" }
      ]
    }
  }
}
```

---

## 八、幂等性

### 8.1 幂等性要求

- GET, PUT, DELETE: 天然幂等
- POST: 敏感操作（如支付）必须幂等

### 8.2 幂等性实现

**方式 1**: 客户端传 Idempotency-Key
```
POST /api/v1/payments
Idempotency-Key: <uuid>
```

服务端记录并返回相同结果。

**方式 2**: 业务层天然幂等
- 如"取消订单"，已取消再调用直接返回成功

---

## 九、版本管理与废弃

### 9.1 版本策略

- 当前版本: `v1`
- 废弃周期: 新版本发布后 6 个月下线旧版本
- 废弃公告: 响应头 `Deprecation: <timestamp>`

### 9.2 破坏性变更

以下被视为破坏性变更，必须升大版本：
- 删除字段
- 修改字段类型
- 修改语义
- 修改错误码

### 9.3 非破坏性变更

可以在小版本内进行：
- 新增字段（可选）
- 新增端点
- 修复 bug

---

## 十、API 文档工具

### 10.1 OpenAPI / Swagger

- 生成方式: [手写 / 代码注解生成]
- 位置: `/api/v1/docs`
- 更新: 每次 API 变更必须同步

### 10.2 Postman 集合

- 位置: 项目根目录 `postman/` 或 `docs/api/`
- 内容: 所有端点的示例请求

---

## 📝 变更日志

| 日期 | 变更 | 关联 ADR |
|------|------|---------|
| YYYY-MM-DD | 初版 | - |
| | | |

---

## 📖 相关文档

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) - 整体架构
- [`SECURITY_DESIGN.md`](./SECURITY_DESIGN.md) - API 安全
- [`DATABASE.md`](./DATABASE.md) - 数据模型

---

> 🎯 **核心提醒**: 
> **API 一旦发布就难以修改**（第三方可能已集成）。
> **设计时多花 1 小时，避免未来 10 小时的向后兼容痛苦。**
