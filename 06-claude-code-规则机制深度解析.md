# Claude Code 规则机制深度解析

## 前言

在前面的笔记中，我们提到了 Claude Code 的记忆系统，其中 `.claude/rules/` 目录作为"模块化规则系统"被简单带过。但实际上，规则机制是 Claude Code 最强大的配置功能之一，它让你能够**精准控制 Claude 在不同场景下的行为**。

想象一下，如果你有一个魔法开关：
- 在处理 API 文件时，自动遵守 RESTful 规范
- 在写测试时，自动应用 TDD 流程
- 在处理安全相关代码时，自动检查 SQL 注入和 XSS 防护
- 在不同目录下，自动切换代码风格

这就是规则机制能做到的！今天咱们就来彻底解析这个"魔法系统"。

---

## 为什么需要规则机制？

### CLAUDE.md 的局限性

CLAUDE.md 是一个很好的起点，但它有几个明显的局限性：

```
┌─────────────────────────────────────────────────────────┐
│  问题 1: 太长了难以维护                                 │
│                                                          │
│  CLAUDE.md 开始时只有 50 行...                            │
│  几个月后变成了 500 行...                                │
│  你: "我刚才写的 API 规则在哪来着？"                        │
│  Claude: "让我在这 500 行里找找..."                       │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  问题 2: 全局适用，不够精准                               │
│                                                          │
│  你在 CLAUDE.md 里写了: "所有函数都要用 snake_case"          │
│                                                          │
│  但在处理 API 文件时，你其实想要 camelCase...              │
│                                                          │
│  Claude: [严格遵守 CLAUDE.md，把 API 函数也改成 snake_case]   │
│  你: [崩溃]                                               │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  问题 3: 无法继承和复用                                   │
│                                                          │
│  每个项目都要重复写同样的安全规则？                       │
│  每个团队都要复制粘贴相同的代码审查清单？                 │
│                                                          │
│  这样的重复劳动太傻了！                                   │
└─────────────────────────────────────────────────────────┘
```

### 规则机制的核心价值

规则机制解决了这些问题，它提供了：

```
┌─────────────────────────────────────────────────────────┐
│  📦 模块化管理                                           │
│  │                                                      │
│  │  把大文件拆成小模块，每个规则文件负责一个领域          │
│  │  - code-style.md (代码风格)                          │
│  │  - security.md (安全规则)                            │
│  │  - testing.md (测试约定)                             │
│  │                                                      │
│  🎯 路径特定规则                                         │
│  │                                                      │
│  │  规则只在匹配的文件路径生效                           │
│  │  "src/api/**/*.ts" → RESTful + camelCase              │
│  │  "src/components/**/*.tsx" → React 最佳实践             │
│  │                                                      │
│  🔄 继承与复用                                           │
│  │                                                      │
│  │  全局规则 → 项目规则 → 路径规则，自动继承与覆盖          │
│  │  团队共享的规则库，一次定义，到处使用                  │
│  │                                                      │
│  📝 版本控制友好                                         │
│  │                                                      │
│  │  每个规则文件独立，修改历史清晰                       │
│  │  便于 Code Review 和团队协作                          │
└─────────────────────────────────────────────────────────┘
```

---

## 规则文件的结构

### 基本组成

每个规则文件都是标准的 Markdown 文件，但有一些特殊的约定：

```markdown
---
# 前置 YAML 元数据（可选，但推荐）
paths:
  - "src/api/**/*.ts"          # 规则适用的文件路径
  - "src/api/**/*.tsx"
priority: 10                   # 优先级（数字越大越优先）
title: "API 开发规范"          # 规则标题
---

# API 开发规范

## 1. 接口命名

- 使用 RESTful 风格
- 路径使用 kebab-case（中横线连接）
- 避免使用动词（HTTP 方法已经表示动作）

❌ 错误示例:
- `/getUser`
- `/user_profile`

✅ 正确示例:
- `GET /users`
- `POST /users`
- `GET /users/{id}`
```

### YAML 元数据详解

**paths**（最强大的功能）：
- 使用 glob 模式匹配文件
- 支持 `**` 表示任意目录深度
- 支持 `*` 表示任意字符序列
- 支持多个路径表达式

```yaml
paths:
  - "src/**/*.test.ts"         # 所有测试文件
  - "**/__mocks__/*.ts"       # 所有 mock 文件
  - "!**/node_modules/**"     # 排除 node_modules
```

**priority**（优先级控制）：
- 数字越大，优先级越高
- 默认优先级：10
- 路径特定规则 > 项目规则 > 全局规则
- 相同路径匹配下，高优先级覆盖低优先级

**title**（规则标题）：
- 可选，但在显示时更友好
- 推荐使用中文或英文，便于识别

---

## 规则的加载顺序

### 三层规则体系

Claude Code 有一个**层级化的规则加载系统**，优先级从高到低：

```
┌─────────────────────────────────────────────────────────┐
│  1. 路径特定规则 (Path-specific rules)                   │
│     └── .claude/rules/ 中带 paths 元数据的文件              │
│     🔥 优先级最高，只在匹配路径时生效                       │
├─────────────────────────────────────────────────────────┤
│  2. 项目规则 (Project rules)                              │
│     └── ./.claude/rules/ 目录下的所有规则文件                │
│     🏠 项目级规范，适用于整个项目                            │
├─────────────────────────────────────────────────────────┤
│  3. 用户规则 (User rules)                                │
│     └── ~/.claude/rules/ 目录下的所有规则文件                │
│     👤 个人偏好，跨项目适用                                │
├─────────────────────────────────────────────────────────┤
│  4. 全局规则 (Global rules)                              │
│     └── /Library/Application Support/ClaudeCode/rules/  (macOS)│
│     🏢 组织级规范，个人无法覆盖                              │
└─────────────────────────────────────────────────────────┘
```

### 覆盖规则

**相同内容，低层级被高层级覆盖**：
- 路径特定规则 > 项目规则 > 用户规则 > 全局规则
- 相同优先级，后加载的覆盖先加载的

**不同内容，自动合并**：
- 不同领域的规则会自动合并
- 例如：路径规则的 API 规范 + 项目规则的代码风格 + 用户规则的测试约定

---

## 规则文件的分类

让我们看看一个典型项目的规则目录结构：

```
.claude/
├── CLAUDE.md               # 主指令文件
└── rules/
    ├── core/               # 核心规则（必选）
    │   ├── code-style.md    # 代码风格
    │   ├── testing.md       # 测试约定
    │   └── security.md      # 安全要求
    ├── domain/             # 业务领域规则（可选）
    │   ├── api-design.md    # API 设计规范
    │   ├── db-access.md     # 数据库操作规范
    │   └── frontend.md      # 前端开发规范
    └── workflow/           # 工作流程规则（可选）
        ├── git-workflow.md  # Git 工作流程
        ├── pr-review.md     # PR 审查清单
        └── ci-cd.md         # CI/CD 规范
```

### 核心规则示例

#### code-style.md（代码风格）

```markdown
---
title: "代码风格规范"
priority: 5
---

# 代码风格规范

## 基本格式

- 使用 2 空格缩进（不要用 Tab）
- 每行不超过 100 字符
- 文件末尾保留一个空行

## 命名约定

### 变量和函数
- 使用 camelCase（小驼峰）
- 变量名要有意义，避免缩写

❌ 错误:
```javascript
let x = 10; // 无意义
let usrNme = "user"; // 不完整的缩写
```

✅ 正确:
```javascript
let userId = 10;
let userName = "user";
```

### 常量
- 使用 UPPER_SNAKE_CASE（大写下划线）
- 只在文件顶部定义

```javascript
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = "https://api.example.com";
```

## 代码注释

- 使用 JSDoc 格式
- 函数必须有注释
- 复杂逻辑必须有解释

```javascript
/**
 * 获取用户信息
 * @param {number} userId - 用户ID
 * @param {boolean} includeDetails - 是否包含详细信息
 * @returns {Promise<User>} 用户信息
 */
async function getUser(userId, includeDetails = false) {
  // ...
}
```
```

#### security.md（安全规则）

```markdown
---
title: "安全开发规范"
priority: 20  # 安全规则优先级最高
---

# 安全开发规范

## 1. SQL 注入防护

❌ 绝对禁止：
```javascript
const sql = `SELECT * FROM users WHERE id = ${userId}`; // 字符串拼接
```

✅ 必须使用参数化查询：
```javascript
// 使用 Knex.js
const user = await knex('users')
  .where('id', userId)
  .first();

// 或者使用 Prisma
const user = await prisma.user.findUnique({
  where: { id: userId }
});
```

## 2. XSS 防护

- 所有用户输入必须进行 HTML 转义
- 避免使用 `innerHTML` 直接插入内容

❌ 错误:
```javascript
document.getElementById('content').innerHTML = userInput;
```

✅ 正确:
```javascript
// 使用 textContent（自动转义）
document.getElementById('content').textContent = userInput;

// 或者使用 React（自动转义）
<div>{userInput}</div>
```

## 3. 密码安全

- 密码必须使用 bcrypt 或 Argon2 哈希
- 禁止存储明文密码
- 密码长度至少 8 位

```javascript
// 使用 bcrypt
import bcrypt from 'bcryptjs';

const saltRounds = 12;
const hashedPassword = await bcrypt.hash('password123', saltRounds);

// 验证密码
const isMatch = await bcrypt.compare('password123', hashedPassword);
```
```

#### testing.md（测试约定）

```markdown
---
title: "测试开发规范"
priority: 10
---

# 测试开发规范

## 1. 测试命名

- 测试文件名：`[原文件名].test.ts`
- 测试函数名：`should_期望行为_when_条件`

```typescript
// math.test.ts
import { sum } from './math';

describe('sum function', () => {
  it('should return 0 when given no numbers', () => {
    expect(sum()).toBe(0);
  });

  it('should return the number when given one number', () => {
    expect(sum(5)).toBe(5);
  });

  it('should return the sum when given multiple numbers', () => {
    expect(sum(1, 2, 3)).toBe(6);
  });
});
```

## 2. 测试结构

每个测试文件应该遵循 AAA 模式：

```typescript
it('should calculate total price', () => {
  // Arrange（准备）
  const items = [
    { name: 'Apple', price: 1.99, quantity: 2 },
    { name: 'Banana', price: 0.99, quantity: 3 }
  ];

  // Act（执行）
  const total = calculateTotal(items);

  // Assert（断言）
  expect(total).toBe(6.95);
});
```

## 3. 测试覆盖率

- 单元测试覆盖率 > 80%
- 核心业务逻辑 > 90%
- 每个 PR 必须保持或提高覆盖率

```bash
# 查看覆盖率报告
npm run test -- --coverage
```
```

---

## 路径特定规则的强大之处

### 场景 1：API 接口开发

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/api/**/*.tsx"
title: "API 开发规范"
priority: 15
---

# API 开发规范

## 1. 接口设计原则

- 使用 RESTful 架构
- 资源名使用复数形式
- HTTP 方法与操作对应：

| 方法 | 操作 | 示例 |
|------|------|------|
| GET | 获取资源 | GET /users |
| POST | 创建资源 | POST /users |
| PUT | 更新资源 | PUT /users/{id} |
| DELETE | 删除资源 | DELETE /users/{id} |

## 2. 响应格式

所有 API 响应必须包含统一的格式：

```typescript
interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  message?: string;
  code?: string;
}
```

✅ 正确示例:
```typescript
// 成功响应
{
  "success": true,
  "data": { "id": 1, "name": "John" },
  "message": "操作成功"
}

// 错误响应
{
  "success": false,
  "message": "用户不存在",
  "code": "USER_NOT_FOUND"
}
```
```

### 场景 2：React 组件开发

```markdown
---
paths:
  - "src/components/**/*.tsx"
  - "src/components/**/*.ts"
title: "React 组件规范"
priority: 12
---

# React 组件规范

## 1. 组件设计

### 函数组件优先
使用函数组件 + Hooks，避免使用 class 组件。

❌ 错误:
```typescript
class UserProfile extends React.Component {
  render() {
    return <div>User Profile</div>;
  }
}
```

✅ 正确:
```typescript
function UserProfile() {
  return <div>User Profile</div>;
}
```

### Props 类型定义
使用 TypeScript 定义 Props 类型。

```typescript
interface UserProfileProps {
  userId: number;
  userName: string;
  avatar?: string; // 可选属性
  onUpdate: (user: User) => void;
}

function UserProfile({ userId, userName, avatar, onUpdate }: UserProfileProps) {
  // ...
}
```

## 2. 状态管理

### 避免过度使用 useState
对于复杂状态，考虑使用 useReducer 或 Context API。

```typescript
// 复杂状态（推荐 useReducer）
type CounterAction = { type: 'increment' } | { type: 'decrement' };

function counterReducer(state: number, action: CounterAction): number {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    default:
      return state;
  }
}

function Counter() {
  const [count, dispatch] = useReducer(counterReducer, 0);

  return (
    <div>
      Count: {count}
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
    </div>
  );
}
```
```

---

## 规则的管理与维护

### 1. 规则的创建

**最佳实践**：
- 从核心规则开始（code-style.md, testing.md, security.md）
- 逐步添加业务领域规则
- 定期审查和更新规则

**工具辅助**：
使用 `/memory` 命令检查当前加载的规则：

```bash
/memory
```

### 2. 规则的测试

如何验证规则是否生效？

```bash
# 1. 检查规则加载情况
/memory

# 2. 测试规则应用
让 Claude 写一个 API 文件，看是否符合规范
让 Claude 写一个组件，看是否应用了 React 最佳实践

# 3. 手动验证
检查生成的代码是否符合规则要求
```

### 3. 规则的更新

规则应该是**活的文档**，需要定期更新：

- 项目技术栈变化时
- 团队规范调整时
- 发现新的最佳实践时

**更新流程**：
1. 评估是否需要更新规则
2. 修改对应的规则文件
3. 通知团队成员
4. 验证规则是否正确应用

---

## 规则机制的高级用法

### 1. 规则的继承与覆盖

**继承**：
```
全局规则 → 项目规则 → 路径规则
↑          ↑          ↑
基础       增强       定制
```

**覆盖**：
- 相同属性，高层级覆盖低层级
- 不同属性，自动合并

### 2. 规则的条件判断

虽然规则主要基于路径匹配，但你可以通过 Markdown 内容实现条件逻辑：

```markdown
---
paths:
  - "*.ts"
  - "*.tsx"
---

# 条件规则

## 1. Node.js 版本检查

如果使用 Node.js 16 或更高版本：

```typescript
// 使用可选链操作符
const userName = user?.name;
```

如果使用 Node.js < 16：

```typescript
// 使用传统方式
const userName = user ? user.name : undefined;
```
```

### 3. 规则的程序化生成

对于复杂项目，你可以使用脚本生成规则文件：

```javascript
// scripts/generate-rules.js
const fs = require('fs');
const path = require('path');

const rulesDir = path.join(__dirname, '.claude', 'rules');

// 从数据库 schema 生成 API 规则
const dbSchema = require('./db-schema.json');
const apiRules = generateApiRules(dbSchema);
fs.writeFileSync(
  path.join(rulesDir, 'api-design.md'),
  apiRules
);

// 从 UI 组件库生成组件规则
const uiLibrary = require('./ui-library.json');
const componentRules = generateComponentRules(uiLibrary);
fs.writeFileSync(
  path.join(rulesDir, 'frontend.md'),
  componentRules
);
```

---

## 常见问题与解决方案

### Q: 规则不生效怎么办？

1. **检查文件路径匹配**：
   - 使用 `/memory` 命令检查规则加载情况
   - 确认文件路径是否符合规则的 paths 模式

2. **检查规则内容**：
   - 确保规则文件格式正确
   - 检查是否有语法错误

3. **检查优先级**：
   - 确认没有其他规则覆盖了当前规则
   - 提高规则的 priority 值（数字越大越优先）

### Q: 规则之间冲突怎么办？

1. **明确优先级**：
   - 给冲突的规则设置不同的优先级
   - 高优先级的规则会覆盖低优先级的

2. **拆分规则**：
   - 更精确地匹配文件路径
   - 避免过于宽泛的 paths 模式

3. **合并规则**：
   - 重新组织规则内容，消除重复和冲突

### Q: 团队成员不遵守规则怎么办？

1. **自动化检查**：
   - 使用 ESLint 或 Prettier 强制执行代码风格
   - 使用 Husky 检查规则是否违反

2. **教育与培训**：
   - 组织规则解读会议
   - 在 PR 评论中引用规则

3. **奖励机制**：
   - 对遵守规则的代码给予积极反馈
   - 评选"规则最佳实践奖"

---

## 规则机制的未来

### 1. 更智能的规则匹配

未来的 Claude Code 可能支持：
- 基于文件内容的规则匹配
- 基于代码复杂度的规则匹配
- 基于开发者经验的规则匹配

### 2. 规则的机器学习

- 自动学习团队的编码风格
- 自动检测和修复规则违规
- 智能推荐规则改进

### 3. 规则的共享生态

- 社区共享的规则库
- 规则的版本管理和发布
- 规则的依赖管理

---

## 总结

规则机制是 Claude Code 最强大的功能之一，它让你能够：

```
┌─────────────────────────────────────────────────────────┐
│  🎯 精准控制 Claude 的行为                               │
│  │                                                      │
│  │  从"全局指令"到"场景化配置"，精确到文件级别              │
│  │                                                      │
│  📦 模块化管理                                           │
│  │                                                      │
│  │  规则文件独立，易于维护和版本控制                      │
│  │                                                      │
│  🔄 继承与复用                                           │
│  │                                                      │
│  │  层级化规则体系，自动合并与覆盖                        │
│  │                                                      │
│  🚀 团队协作增强                                         │
│  │                                                      │
│  │  统一的规范，减少沟通成本，提高开发效率                  │
│  │                                                      │
│  💡 最佳实践固化                                         │
│  │                                                      │
│  │  把团队的知识和经验转化为可执行的规则                  │
│  │  新人快速上手，老人避免重复踩坑                        │
└─────────────────────────────────────────────────────────┘
```

**一句话总结**：规则机制让 Claude Code 从"通用助手"变成了"你的专属专家"。

**建议**：
1. 从小规模开始，先建立核心规则
2. 逐步完善，根据项目需要添加新规则
3. 定期审查和更新，保持规则的时效性
4. 充分利用路径特定规则，实现精准配置

---

**参考文档**：[Rules - Claude Code Docs](https://code.claude.com/docs/en/rules)
