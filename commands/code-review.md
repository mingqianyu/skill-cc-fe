<!--
  【设计理念】
  /code-review 是开源项目中使用频率最高的命令。
  核心思想：写完代码后立即审查，不要等到 PR 阶段。

  【与 agent 的关系】
  此 command 调用 code-reviewer agent 执行实际审查。
  command 定义审查流程和输出格式，
  agent 定义审查能力和专业知识。
-->

# /code-review 命令

对未提交的代码变更进行全面的安全和质量审查。

## 执行流程

1. 获取变更文件: `git diff --name-only HEAD`

2. 对每个变更文件检查：

**安全问题 (CRITICAL):**
- 硬编码凭证、API keys、tokens
- SQL 注入漏洞
- XSS 漏洞
- 缺少输入校验
- 不安全的依赖
- 路径穿越风险

**代码质量 (HIGH):**
- 函数 > 50 行
- 文件 > 800 行
- 嵌套深度 > 4 层
- 缺少错误处理
- console.log 语句
- 缺少测试

**最佳实践 (MEDIUM):**
- 可变数据修改（应用不可变模式）
- 缺少 TypeScript 类型
- 可访问性问题

3. 生成报告：
   - 严重级别: CRITICAL, HIGH, MEDIUM, LOW
   - 文件位置和行号
   - 问题描述
   - 修复建议

4. 存在 CRITICAL 或 HIGH 问题时阻止提交

**绝不放过安全漏洞！**
