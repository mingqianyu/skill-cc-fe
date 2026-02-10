---
name: coding-standards
description: TypeScript 编码标准、命名规范、代码组织最佳实践
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 coding-standards skill。
  定义 TypeScript 项目的编码标准和命名规范。

  【编码标准的核心原则】
  开源项目总结了 4 个优先级：
  1. 可读性优先（Readability First）
  2. KISS（Keep It Simple, Stupid）
  3. DRY（Don't Repeat Yourself）
  4. YAGNI（You Aren't Gonna Need It）
-->

# TypeScript 编码标准

## 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量/函数 | camelCase | `userName`, `getUserList` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 类/接口/类型 | PascalCase | `UserInfo`, `ApiResponse` |
| 组件文件 | PascalCase | `UserCard.vue` |
| 工具文件 | camelCase | `formatDate.ts` |
| 目录 | kebab-case | `user-management/` |

## 函数命名约定

```typescript
// 获取数据：get / fetch / query
function getUserById(id: string) {}
function fetchOrderList() {}

// 设置数据：set / update
function setUserName(name: string) {}
function updateOrderStatus(id: string, status: string) {}

// 判断/检查：is / has / can / should
function isAdmin(user: User): boolean {}
function hasPermission(role: string): boolean {}

// 事件处理：handle + 事件名
function handleSubmit() {}
function handlePageChange(page: number) {}

// 转换/格式化：to / format / parse
function toFormData(obj: Record<string, any>) {}
function formatPrice(cents: number): string {}
```

## TypeScript 类型规范

```typescript
// ✅ 优先使用 interface 定义对象类型
interface UserInfo {
  id: string
  name: string
  email: string
  role: 'admin' | 'user'
}

// ✅ 使用 type 定义联合类型、交叉类型
type Status = 'pending' | 'active' | 'disabled'
type ApiResult<T> = { success: true; data: T } | { success: false; error: string }

// ❌ 禁止使用 any
function process(data: any) {} // 不要这样

// ✅ 使用 unknown 替代 any
function process(data: unknown) {
  if (typeof data === 'string') {
    // 类型收窄后安全使用
  }
}
```

## 代码异味检测

以下情况需要重构：
- 函数超过 50 行
- 嵌套超过 4 层
- 魔法数字（未命名的常量）
- 重复代码块
- 过长的参数列表（> 3 个参数用对象）
