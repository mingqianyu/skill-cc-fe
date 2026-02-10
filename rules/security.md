<!--
  【设计理念】
  安全规则。参考开源项目的 security.md。

  【为什么安全规则是最高优先级？】
  开源项目将安全检查标记为 CRITICAL 级别，原因：
  1. 安全漏洞一旦上线，修复成本极高
  2. Claude 生成代码时可能无意中引入安全问题
  3. 硬编码密钥、SQL 注入、XSS 是最常见的安全隐患
  4. 通过规则约束，让 Claude 在生成代码时就避免这些问题

  【安全响应协议】
  开源项目定义了发现安全问题时的标准流程：
  立即停止 → 分析 → 修复 → 排查同类问题
-->

# 安全规则

## 提交前必检清单 (CRITICAL)

每次提交前必须确认：

- [ ] 无硬编码密钥（API keys、密码、tokens）
- [ ] 所有用户输入已校验
- [ ] SQL 注入防护（参数化查询）
- [ ] XSS 防护（转义 HTML 输出）
- [ ] CSRF 防护已启用
- [ ] 鉴权/授权已验证
- [ ] API 接口有限流保护
- [ ] 错误信息不泄露敏感数据

## 密钥管理

```typescript
// ❌ 禁止：硬编码密钥
const apiKey = "sk-proj-xxxxx"
const dbPassword = "root123456"

// ✅ 正确：使用环境变量
const apiKey = process.env.API_KEY
const dbPassword = process.env.DB_PASSWORD

// ✅ 启动时检查必要的环境变量
if (!process.env.JWT_SECRET) {
  throw new Error('JWT_SECRET 环境变量未配置')
}
```

## 前端安全

```typescript
// ❌ XSS 风险：直接插入 HTML
element.innerHTML = userInput

// ✅ 使用 Vue 的模板系统（自动转义）
// <span>{{ userInput }}</span>

// ❌ 禁止在 URL 中拼接用户输入
window.location.href = `/page?redirect=${userInput}`

// ✅ 使用 encodeURIComponent
window.location.href = `/page?redirect=${encodeURIComponent(userInput)}`
```

## 后端安全

```typescript
// ❌ SQL 注入风险
const query = `SELECT * FROM users WHERE name = '${userName}'`

// ✅ 参数化查询
const [rows] = await pool.execute(
  'SELECT * FROM users WHERE name = ?',
  [userName]
)

// ✅ 限流中间件
import rateLimit from 'express-rate-limit'
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100 // 每个 IP 最多 100 次请求
})
app.use('/api/', limiter)
```

## 安全响应协议

发现安全问题时：

1. **立即停止** 当前工作
2. 使用 **security-reviewer** 代理分析
3. 先修复 CRITICAL 级别问题
4. 轮换已暴露的密钥
5. 全库排查同类问题
