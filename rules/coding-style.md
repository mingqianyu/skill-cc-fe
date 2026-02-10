<!--
  【设计理念】
  Rules 是 Claude Code 中"始终生效"的规则文件。
  放在 rules/ 目录下的 .md 文件会被 Claude 自动加载并遵循。

  与 Skills 的区别：
  - Rules = 强制规则，每次对话都会加载（类似 ESLint 配置）
  - Skills = 按需知识，只在相关任务时引用（类似技术文档）

  【为什么用 Markdown？】
  Claude 是大语言模型，自然语言是它最擅长理解的格式。
  用 .md 写规则比用 JSON/YAML 配置更灵活、更易维护。
  Claude 能理解上下文、例外情况和"为什么"，而不只是机械执行。

  【本文件】编码风格规则。
  定义不可变数据、文件组织、错误处理、输入校验等基础编码规范。
-->

# 编码风格规则

## 不可变性 (CRITICAL)

<!--
  【为什么不可变性是 CRITICAL？】
  可变数据是 Bug 的主要来源之一：
  1. Vue 3 的响应式系统依赖 Proxy，直接修改嵌套对象可能导致响应丢失
  2. Pinia store 中直接修改 state 会绕过 devtools 追踪
  3. 不可变数据更容易推理、测试和调试
-->

始终创建新对象，禁止直接修改已有对象：

```typescript
// ❌ 错误：直接修改对象（mutation）
function updateUser(user: User, name: string) {
  user.name = name  // 直接修改了原对象！
  return user
}

// ✅ 正确：创建新对象（immutable）
function updateUser(user: User, name: string): User {
  return { ...user, name }
}

// ❌ 错误：直接修改数组
function addItem(list: Item[], item: Item) {
  list.push(item)  // 修改了原数组！
  return list
}

// ✅ 正确：创建新数组
function addItem(list: Item[], item: Item): Item[] {
  return [...list, item]
}
```

### Vue 3 中的不可变性

```typescript
// ❌ 错误：在 Pinia store 中直接修改嵌套对象
store.user.profile.name = 'new name'

// ✅ 正确：使用 $patch 或 action
store.$patch({
  user: { ...store.user, profile: { ...store.user.profile, name: 'new name' } }
})

// ✅ 更好：在 action 中处理
// store/user.ts
actions: {
  updateProfileName(name: string) {
    this.user = { ...this.user, profile: { ...this.user.profile, name } }
  }
}
```

## 文件组织

<!--
  【设计理念】"多小文件"优于"少大文件"
  这是开源项目反复强调的原则。原因：
  1. Claude 的上下文窗口有限，小文件更容易完整读取
  2. 小文件职责单一，修改时影响范围小
  3. 代码审查时更容易理解变更
  4. Git 冲突概率更低
-->

**多小文件 > 少大文件**：

- 单文件 200-400 行为宜，最多不超过 800 行
- 高内聚、低耦合
- 按功能/领域组织，而非按类型组织
- 大组件拆分为子组件 + 组合式函数

```
# ❌ 按类型组织（不推荐）
src/
├── components/    # 所有组件混在一起
├── stores/        # 所有 store 混在一起
└── utils/         # 所有工具函数混在一起

# ✅ 按功能/领域组织（推荐）
src/
├── features/
│   ├── user/
│   │   ├── components/    # 用户相关组件
│   │   ├── composables/   # 用户相关组合式函数
│   │   ├── stores/        # 用户 store
│   │   ├── types/         # 用户类型定义
│   │   └── api/           # 用户 API
│   └── order/
│       ├── components/
│       ├── composables/
│       └── ...
```

## 错误处理

<!--
  【为什么错误处理是必须的？】
  未处理的错误会导致：
  1. 前端白屏或功能失效，用户体验极差
  2. 后端接口返回 500，客户端无法理解错误原因
  3. 调试困难，错误被"吞掉"后无法追踪
-->

始终全面处理错误：

```typescript
// 前端 API 调用
async function fetchUserList(): Promise<User[]> {
  try {
    const { data } = await api.get<ApiResponse<User[]>>('/users')
    return data.data ?? []
  } catch (error) {
    // 记录错误用于调试
    console.error('获取用户列表失败:', error)
    // 向用户展示友好的错误信息
    ElMessage.error('获取用户列表失败，请稍后重试')
    return []
  }
}

// Node.js 后端
async function getUserById(req: Request, res: Response) {
  try {
    const user = await userService.findById(req.params.id)
    if (!user) {
      return res.status(404).json({ success: false, error: '用户不存在' })
    }
    res.json({ success: true, data: user })
  } catch (error) {
    console.error('查询用户失败:', error)
    // 不要向客户端暴露内部错误细节
    res.status(500).json({ success: false, error: '服务器内部错误' })
  }
}
```

## 输入校验

<!--
  【设计理念】在系统边界做校验
  开源项目强调"只在系统边界验证"——即用户输入、外部 API 返回值。
  内部函数之间的调用可以信任类型系统，不需要重复校验。
-->

所有用户输入必须校验：

```typescript
// 推荐使用 zod 做 schema 校验（类型安全 + 运行时校验）
import { z } from 'zod'

// 定义校验 schema
const createUserSchema = z.object({
  username: z.string().min(2).max(20),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional()
})

// 在 API 入口处校验
app.post('/api/users', (req, res) => {
  const result = createUserSchema.safeParse(req.body)
  if (!result.success) {
    return res.status(400).json({
      success: false,
      error: '参数校验失败',
      details: result.error.flatten()
    })
  }
  // result.data 已经是类型安全的
  userService.create(result.data)
})
```

## 代码质量自检清单

<!--
  【设计理念】完成前自检
  开源项目要求在标记任务完成前，必须过一遍自检清单。
  这不是形式主义，而是 Claude 容易遗漏的点的提醒。
-->

在标记任务完成前，检查以下项目：

- [ ] 代码可读性好，命名清晰
- [ ] 函数短小（< 50 行）
- [ ] 文件聚焦（< 800 行）
- [ ] 无深层嵌套（> 4 层）
- [ ] 有适当的错误处理
- [ ] 无 console.log（生产代码）
- [ ] 无硬编码值（使用常量或环境变量）
- [ ] 使用不可变模式（无直接修改）
- [ ] TypeScript 类型完整，无 any
