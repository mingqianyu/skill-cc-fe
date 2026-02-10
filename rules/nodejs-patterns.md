<!--
  【设计理念】
  Node.js 后端开发模式规则。
  参考开源项目的 backend-patterns skill，适配为 Express + MySQL/PostgreSQL 技术栈。

  【为什么需要后端规则？】
  全栈项目中，前后端代码风格应保持一致。
  Claude 需要知道后端的分层架构、错误处理、数据库操作等约定，
  才能生成符合团队规范的后端代码。
-->

# Node.js 后端开发模式规则

## 分层架构

<!--
  【设计理念】经典三层架构
  开源项目推荐 Repository Pattern + Service Layer + Middleware Pattern。
  这种分层让每一层职责清晰：
  - Router: 路由定义和参数提取
  - Controller: 请求/响应处理
  - Service: 业务逻辑
  - Model/DAO: 数据访问
-->

```
src/
├── routes/          # 路由定义
├── controllers/     # 控制器（处理请求/响应）
├── services/        # 业务逻辑层
├── models/          # 数据模型 / DAO
├── middlewares/      # 中间件（鉴权、日志、错误处理）
├── utils/           # 工具函数
├── types/           # TypeScript 类型定义
├── config/          # 配置文件
└── app.ts           # 应用入口
```

## API 响应格式

<!--
  【设计理念】统一响应格式
  开源项目定义了标准的 ApiResponse 接口。
  统一格式的好处：
  1. 前端可以写通用的响应拦截器
  2. 错误处理逻辑一致
  3. 分页信息标准化
-->

```typescript
// types/api.ts
interface ApiResponse<T = any> {
  success: boolean
  data?: T
  error?: string
  message?: string
  meta?: {
    total: number
    page: number
    pageSize: number
  }
}

// 成功响应
res.json({ success: true, data: result })

// 分页响应
res.json({
  success: true,
  data: list,
  meta: { total: 100, page: 1, pageSize: 10 }
})

// 错误响应
res.status(400).json({ success: false, error: '参数错误' })
```

## 中间件模式

```typescript
// middlewares/auth.ts - 鉴权中间件
export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '')
  if (!token) {
    return res.status(401).json({ success: false, error: '未登录' })
  }
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!)
    req.user = decoded as UserPayload
    next()
  } catch {
    res.status(401).json({ success: false, error: 'Token 已过期' })
  }
}

// middlewares/errorHandler.ts - 全局错误处理
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  console.error('服务器错误:', err)
  res.status(500).json({
    success: false,
    error: process.env.NODE_ENV === 'production' ? '服务器内部错误' : err.message
  })
}
```

## 数据库操作规范

```typescript
// ✅ 使用参数化查询，防止 SQL 注入
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE id = ? AND status = ?',
  [userId, 'active']
)

// ❌ 禁止字符串拼接 SQL
const rows = await pool.query(`SELECT * FROM users WHERE id = ${userId}`)

// ✅ 事务处理
const connection = await pool.getConnection()
try {
  await connection.beginTransaction()
  await connection.execute('UPDATE accounts SET balance = balance - ? WHERE id = ?', [amount, fromId])
  await connection.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?', [amount, toId])
  await connection.commit()
} catch (error) {
  await connection.rollback()
  throw error
} finally {
  connection.release()
}
```

## 环境变量管理

```typescript
// config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number).default('3000'),
  DB_HOST: z.string(),
  DB_PORT: z.string().transform(Number),
  DB_USER: z.string(),
  DB_PASSWORD: z.string(),
  DB_NAME: z.string(),
  JWT_SECRET: z.string().min(32),
})

// 启动时校验环境变量，缺失则立即报错
export const env = envSchema.parse(process.env)
```
