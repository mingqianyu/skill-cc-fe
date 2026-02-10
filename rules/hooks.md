<!--
  【设计理念】
  钩子系统说明。参考开源项目的 hooks.md。

  【什么是 Hooks？】
  Hooks 是 Claude Code 的自动化机制：
  - PreToolUse: 工具执行前触发（校验、拦截）
  - PostToolUse: 工具执行后触发（格式化、检查）
  - Stop: Claude 响应结束时触发（最终检查）
  - SessionStart/End: 会话开始/结束时触发
  - PreCompact: 上下文压缩前触发

  【为什么 Hooks 很重要？】
  Hooks 实现了"自动化质量保障"：
  - 编辑 .ts 文件后自动运行 TypeScript 检查
  - 编辑代码后自动格式化
  - 提交前自动检查 console.log
  - 阻止创建不必要的文档文件
  这些都不需要人工干预，大幅减少低级错误。
-->

# 钩子系统说明

## 钩子类型

| 类型 | 触发时机 | 用途 |
|------|----------|------|
| PreToolUse | 工具执行前 | 校验、拦截、提醒 |
| PostToolUse | 工具执行后 | 格式化、检查、反馈 |
| Stop | 响应结束时 | 最终验证 |
| SessionStart | 会话开始 | 加载上下文 |
| SessionEnd | 会话结束 | 保存状态 |
| PreCompact | 压缩前 | 保存关键信息 |

## 当前配置的钩子

详见 `hooks/hooks.json`，主要包括：

### PreToolUse
- 长耗时命令提醒使用 tmux
- push 前提醒审查
- 阻止创建不必要的 .md 文件

### PostToolUse
- 编辑 JS/TS 文件后自动 Prettier 格式化
- 编辑 .ts/.tsx 后运行 TypeScript 检查
- 检测 console.log 并警告

### Stop
- 检查所有修改文件中的 console.log

## 自动通过权限

使用需谨慎：
- 仅在可信、计划明确时开启
- 探索性工作应关闭
- 禁止使用 `dangerously-skip-permissions`
