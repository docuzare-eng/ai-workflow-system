# 测试策略 (Test Strategy)

> 本文件是 R6 QA 工程师的核心产出。定义项目的测试方法论。

---

## 📌 元信息

| 字段 | 值 |
|------|-----|
| 项目代号 | `[项目代号]` |
| 文档版本 | `v1.0` |
| 最后更新 | `YYYY-MM-DD` |

---

## 一、测试金字塔

```
        /\
       /E2E\       5-10%   慢、贵、脆弱，只测核心旅程
      /─────\
     /集成测试\    20-30%  覆盖模块间协作
    /─────────\
   /  单元测试  \  60-70%  快、稳、多
  /─────────────\
```

### 数量分布目标

| 类型 | 数量预估 | 执行时间 |
|------|---------|---------|
| 单元测试 | [X] 个 | < 30 秒 |
| 集成测试 | [X] 个 | < 2 分钟 |
| E2E 测试 | [X] 个 | < 5 分钟 |

---

## 二、测试覆盖目标

### 2.1 覆盖率目标

| 级别 | 目标 | 范围 |
|------|------|------|
| **整体** | ≥ 70% | 全项目 |
| **核心模块** | ≥ 85% | auth, core domains |
| **工具函数** | ≥ 90% | utils |
| **UI 组件** | ≥ 60% | 关键交互组件 |

### 2.2 必须覆盖的场景

**每个用户故事**:
- ✅ Happy Path
- ✅ 至少 2 条错误路径
- ✅ 所有验收标准

**每个 API 端点**:
- ✅ 成功响应
- ✅ 认证失败
- ✅ 授权失败
- ✅ 输入验证失败
- ✅ 业务规则失败

---

## 三、测试类型

### 3.1 单元测试 (Unit Tests)

**工具**: Vitest

**测什么**:
- 纯函数
- 业务服务
- 工具函数
- Hooks
- 组件（简单的）

**原则**:
- 独立：不依赖外部
- 快速：毫秒级
- 确定性：不依赖时间/随机

**示例**:
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Given
      const input = { email: 'test@example.com', password: '...' };
      
      // When
      const user = await userService.createUser(input);
      
      // Then
      expect(user.email).toBe('test@example.com');
    });
    
    it('should throw error for duplicate email', async () => {
      // ...
    });
  });
});
```

### 3.2 集成测试 (Integration Tests)

**工具**: Vitest + supertest

**测什么**:
- API 端点（含数据库）
- 跨模块流程
- 与真实数据库的交互

**环境**:
- 独立测试数据库
- 每个测试自清理
- 真实 HTTP 请求

### 3.3 E2E 测试

**工具**: Playwright

**测什么**:
- 核心用户旅程（见 USER_JOURNEYS.md）
- 关键功能完整流程
- 跨浏览器兼容

**运行环境**:
- Preview 环境
- 生产环境（只读测试）

### 3.4 性能测试

**工具**: [k6 / Artillery]

**场景**:
- 登录压测: [目标 QPS]
- 核心 API 压测
- 长时间稳定性测试

### 3.5 安全测试

**手段**:
- 依赖扫描（npm audit, CI 自动）
- SAST（静态代码分析）
- DAST（动态扫描）
- 渗透测试（重要项目）

---

## 四、测试数据管理

### 4.1 数据策略

- **单元测试**: 内存数据 / Mock
- **集成测试**: 独立 DB + Factory 模式
- **E2E 测试**: 种子数据 + 测试账号

### 4.2 Factory 模式

```typescript
const UserFactory = {
  build: (overrides?: Partial<User>) => ({
    email: faker.internet.email(),
    name: faker.person.fullName(),
    ...overrides
  }),
  create: async (overrides?: Partial<User>) => {
    return db.user.create({ data: UserFactory.build(overrides) });
  }
};
```

### 4.3 清理策略

- 每个测试前: 清空相关表
- 每个测试后: 不需要（下次重置）
- 测试数据隔离: 使用 transaction + rollback

---

## 五、测试环境

### 5.1 环境矩阵

| 环境 | 用途 | 数据 | 自动化 |
|------|------|------|--------|
| Local | 开发 | 本地种子 | 手动 |
| CI | 自动验证 | 测试种子 | 每次 PR |
| Preview | 集成验证 | 隔离 DB | 部署后 |
| Staging | 预发验证 | 类生产 | 上线前 |

### 5.2 CI 触发

```yaml
# 每次 PR:
- 单元测试
- 集成测试
- Lint + TypeCheck

# 合并到 main:
- 上述 +
- E2E 测试
- 部署到 staging

# 每日:
- 完整回归测试
- 性能测试
```

---

## 六、测试用例管理

### 6.1 测试用例清单

每个功能模块维护测试用例清单：

```markdown
# 用户认证模块测试用例

## TC-AUTH-001: 用户注册
- **关联**: US-001
- **类型**: 单元 + 集成 + E2E
- **场景**:
  - TC-001-001: 成功注册
  - TC-001-002: 邮箱已存在
  - TC-001-003: 密码不符合强度
  - TC-001-004: 格式错误
```

### 6.2 缺陷追踪

发现 bug 时创建 `BUG-XXX.md` 记录：
- 现象
- 重现步骤
- 期望 vs 实际
- 影响级别
- 修复验证

---

## 七、质量门槛

### PR 合并门槛

- [ ] 所有测试通过
- [ ] 覆盖率不下降
- [ ] Lint 无错误
- [ ] TypeCheck 无错误
- [ ] 至少 1 人审查通过

### 发布门槛

- [ ] 所有自动化测试通过
- [ ] 手动测试清单完成
- [ ] 安全扫描无高危
- [ ] 性能指标达标
- [ ] 回滚方案确认

---

## 📖 相关文档

- [`../03-requirements/USER_JOURNEYS.md`](../03-requirements/USER_JOURNEYS.md)
- [`SECURITY_AUDIT.md`](./SECURITY_AUDIT.md)
- [`PERFORMANCE_REPORT.md`](./PERFORMANCE_REPORT.md)

---

> 🎯 **核心提醒**: 
> **没有测试保护的代码，就是定时炸弹。**
> **今天偷懒不写测试 = 明天在凌晨 3 点改 bug。**
