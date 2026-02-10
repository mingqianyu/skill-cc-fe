---
name: security-review
description: 安全审查技能 - OWASP Top 10 漏洞检测、前后端安全最佳实践
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 security-review skill。

  【Security Skill vs Security Rule vs Security Agent】
  - security rule: 简洁的安全检查清单（始终加载）
  - security skill: 详细的安全知识和示例（按需引用）
  - security agent: 执行安全审查的专家角色

  【为什么安全需要三层？】
  rule 确保 Claude 始终记住安全底线，
  skill 提供深度知识让 Claude 知道怎么修，
  agent 在需要时执行专项安全审查。
-->

# 安全审查技能

## OWASP Top 10 检查

### 1. 注入攻击

```typescript
// ❌ SQL 注入
const query = `SELECT * FROM users WHERE name = '${name}'`

// ✅ 参数化查询
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE name = ?', [name]
)
```

### 2. XSS 跨站脚本

```vue
<!-- ❌ 直接渲染 HTML -->
<div v-html="userInput"></div>

<!-- ✅ 使用文本插值（自动转义） -->
<div>{{ userInput }}</div>

<!-- 如果必须渲染 HTML，先消毒 -->
<div v-html="sanitize(userInput)"></div>
```

### 3. 敏感数据泄露

```typescript
// ❌ 返回完整用户对象（包含密码）
res.json({ success: true, data: user })

// ✅ 排除敏感字段
const { password, ...safeUser } = user
res.json({ success: true, data: safeUser })
```

### 4. CSRF 防护

```typescript
// Express 中使用 csurf 中间件
import csrf from 'csurf'
app.use(csrf({ cookie: true }))
```

## 前端安全清单

- [ ] 不在前端存储敏感信息（密码、密钥）
- [ ] Token 存储在 httpOnly cookie 中
- [ ] 所有外部链接加 `rel="noopener noreferrer"`
- [ ] 用户输入不直接拼接到 URL 中
- [ ] 不使用 `eval()` 或 `new Function()`
