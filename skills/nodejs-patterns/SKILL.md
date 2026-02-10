---
name: nodejs-patterns
description: Node.js + Express 后端开发模式、中间件、数据库操作最佳实践
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 backend-patterns skill，
  适配为 Express + MySQL/PostgreSQL 技术栈。

  【Skills 的深度 vs Rules 的广度】
  rules/nodejs-patterns.md 定义了"必须遵守的规范"（简洁）
  这个 skill 提供了"怎么做得好"的详细示例（深入）
-->

# Node.js + Express 后端模式

## 服务层模式（Service Layer）

```typescript
// services/userService.ts
import { pool } from '@/config/database'
import type { User, CreateUserDto } from '@/types/user'

export const userService = {
  async findById(id: string): Promise<User | null> {
    const [rows] = await pool.execute(
      'SELECT * FROM users WHERE id = ? AND deleted_at IS NULL',
      [id]
    )
    return (rows as User[])[0] ?? null
  },

  async create(dto: CreateUserDto): Promise<User> {
    const { username, email, password } = dto
    const hashedPassword = await bcrypt.hash(password, 10)
    const [result] = await pool.execute(
      'INSERT INTO users (username, email, password) VALUES (?, ?, ?)',
      [username, email, hashedPassword]
    )
    return this.findById((result as any).insertId)!
  },

  async findByPage(page: number, pageSize: number) {
    const offset = (page - 1) * pageSize
    const [rows] = await pool.execute(
      'SELECT * FROM users WHERE deleted_at IS NULL LIMIT ? OFFSET ?',
      [pageSize, offset]
    )
    const [countResult] = await pool.execute(
      'SELECT COUNT(*) as total FROM users WHERE deleted_at IS NULL'
    )
    return {
      list: rows as User[],
      total: (countResult as any)[0].total
    }
  }
}
```

## 中间件模式

```typescript
// middlewares/validate.ts
import { z, ZodSchema } from 'zod'

// 通用请求校验中间件
export function validate(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body)
    if (!result.success) {
      return res.status(400).json({
        success: false,
        error: '参数校验失败',
        details: result.error.flatten()
      })
    }
    req.body = result.data
    next()
  }
}

// 使用示例
const createUserSchema = z.object({
  username: z.string().min(2).max(20),
  email: z.string().email()
})

router.post('/users', validate(createUserSchema), userController.create)
```

## 错误处理模式

```typescript
// utils/AppError.ts
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
  }
}

// 使用
throw new AppError(404, '用户不存在')
throw new AppError(403, '无权限访问')
```
