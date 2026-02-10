---
name: backend-patterns
description: 后端通用模式 - API 设计、缓存策略、认证授权、限流
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 backend-patterns skill。
  与 nodejs-patterns skill 的区别：
  - nodejs-patterns: Express 框架特有的模式
  - backend-patterns: 通用后端设计模式（缓存、认证、限流等）
-->

# 后端通用模式

## RESTful API 设计

```
GET    /api/users          # 获取用户列表
GET    /api/users/:id      # 获取单个用户
POST   /api/users          # 创建用户
PUT    /api/users/:id      # 更新用户（全量）
PATCH  /api/users/:id      # 更新用户（部分）
DELETE /api/users/:id      # 删除用户
```

## 统一响应格式

```typescript
// 成功
{ success: true, data: { ... } }

// 分页
{ success: true, data: [...], meta: { total, page, pageSize } }

// 错误
{ success: false, error: '错误描述' }
```

## JWT 认证模式

```typescript
// 生成 token
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET!,
  { expiresIn: '7d' }
)

// 验证 token（中间件）
const decoded = jwt.verify(token, process.env.JWT_SECRET!)
```

## 缓存策略

```typescript
// Cache-Aside 模式
async function getUserById(id: string) {
  // 1. 先查缓存
  const cached = await redis.get(`user:${id}`)
  if (cached) return JSON.parse(cached)

  // 2. 缓存未命中，查数据库
  const user = await db.query('SELECT * FROM users WHERE id = ?', [id])

  // 3. 写入缓存（设置过期时间）
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user))

  return user
}
```
