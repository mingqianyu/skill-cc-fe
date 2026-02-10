# fe-skill - Vue 3 + Node.js 全栈开发技能包

> 基于 [everything-claude-code](https://github.com/affaan-m/everything-claude-code) 开源项目的设计理念，
> 为 Vue 3 + TypeScript + Node.js 全栈开发定制的 Claude Code 技能包。

## 设计理念

### 1. 为什么需要技能包？

Claude Code 本身是一个强大的 AI 编程助手，但它缺乏**项目特定的上下文**。
技能包的核心价值在于：

- **减少重复沟通**：将团队规范、编码风格、架构模式固化为规则
- **提高代码一致性**：Claude 每次生成的代码都遵循相同的标准
- **降低 Bug 率**：通过自动化检查（hooks）在编码时就发现问题
- **标准化工作流**：plan → tdd → code-review → verify 的完整流程

### 2. 核心设计原则（来自开源项目）

| 原则 | 说明 |
|------|------|
| **声明式配置** | 通过 .md 文件描述规则，Claude 自然语言理解 |
| **模块化分离** | skills/agents/commands/rules/hooks 各司其职 |
| **渐进式采用** | 可以只用 rules，也可以启用完整工作流 |
| **AI 友好** | 所有规则用自然语言编写，Claude 能直接理解和执行 |

### 3. 目录结构说明

```
fe-skill/
├── .claude-plugin/          # 插件入口配置
│   └── plugin.json          # 声明 skills、agents 路径
├── agents/                  # 子代理（专家角色）
│   ├── planner.md           # 规划专家 - 复杂功能拆解
│   ├── code-reviewer.md     # 代码审查 - 质量和安全检查
│   ├── architect.md         # 架构师 - 系统设计决策
│   ├── vue-reviewer.md      # Vue 专项审查 - Vue 3 最佳实践
│   ├── build-error-resolver.md  # 构建错误修复专家
│   ├── security-reviewer.md # 安全审查专家
│   ├── tdd-guide.md         # TDD 工作流指导
│   ├── refactor-cleaner.md  # 重构和死代码清理
│   └── doc-updater.md       # 文档同步更新
├── commands/                # 用户可调用的命令（/command）
│   ├── plan.md              # /plan - 创建实施计划
│   ├── code-review.md       # /code-review - 代码审查
│   ├── tdd.md               # /tdd - 测试驱动开发
│   ├── build-fix.md         # /build-fix - 修复构建错误
│   ├── verify.md            # /verify - 综合验证
│   ├── checkpoint.md        # /checkpoint - 检查点管理
│   ├── refactor-clean.md    # /refactor-clean - 重构清理
│   ├── test-coverage.md     # /test-coverage - 测试覆盖率
│   ├── orchestrate.md       # /orchestrate - 多代理编排
│   ├── learn.md             # /learn - 提取可复用模式
│   └── update-docs.md       # /update-docs - 更新文档
├── contexts/                # 上下文模式（切换工作模式）
│   ├── dev.md               # 开发模式 - 写代码优先
│   ├── review.md            # 审查模式 - 质量优先
│   └── research.md          # 研究模式 - 理解优先
├── hooks/                   # 自动化钩子
│   └── hooks.json           # 钩子配置（编辑后自动格式化等）
├── rules/                   # 开发规则（始终生效）
│   ├── coding-style.md      # 编码风格规范
│   ├── vue-patterns.md      # Vue 3 开发模式
│   ├── nodejs-patterns.md   # Node.js 后端模式
│   ├── git-workflow.md      # Git 工作流规范
│   ├── security.md          # 安全规则
│   ├── testing.md           # 测试规范
│   ├── performance.md       # 性能优化规则
│   ├── agents.md            # 代理编排规则
│   └── hooks.md             # 钩子系统说明
├── skills/                  # 技能定义（深度知识）
│   ├── vue3-patterns/       # Vue 3 组件和组合式 API 模式
│   ├── uniapp-patterns/     # uni-app 跨端开发模式
│   ├── nodejs-patterns/     # Node.js + Express 后端模式
│   ├── coding-standards/    # 编码标准（TypeScript 规范）
│   ├── frontend-patterns/   # 前端通用模式
│   ├── backend-patterns/    # 后端通用模式
│   ├── tdd-workflow/        # TDD 工作流技能
│   ├── verification-loop/   # 验证循环技能
│   └── security-review/     # 安全审查技能
├── examples/                # 示例文件
│   └── CLAUDE.md            # 项目级 CLAUDE.md 示例
└── README.md                # 本文件
```

## 核心概念解析

### Skills vs Commands vs Agents vs Rules

这是开源项目最核心的设计理念——**关注点分离**：

| 概念 | 作用 | 触发方式 | 类比 |
|------|------|----------|------|
| **Rules** | 始终生效的规则 | 自动加载 | 公司规章制度 |
| **Skills** | 深度领域知识 | 按需引用 | 专业技能手册 |
| **Commands** | 用户触发的工作流 | `/command` 调用 | 标准操作流程(SOP) |
| **Agents** | 专家角色子代理 | 被 commands 调用 | 专家顾问 |
| **Hooks** | 自动化检查 | 工具执行前后触发 | 质检流水线 |
| **Contexts** | 工作模式切换 | 手动切换 | 工作场景 |

### 工作流设计（开源项目的核心流程）

```
用户需求
  │
  ▼
/plan（规划）──→ planner agent 分析需求、拆解步骤
  │
  ▼
/tdd（开发）──→ tdd-guide agent 先写测试再实现
  │
  ▼
/code-review ──→ code-reviewer agent 审查代码质量
  │
  ▼
/verify（验证）──→ 构建、类型检查、lint、测试全部通过
  │
  ▼
/checkpoint ──→ 创建检查点，记录进度
```

## 技术栈

- **前端**: Vue 3 + TypeScript + Pinia + Vue Router
- **跨端**: uni-app (小程序/H5/App)
- **UI 库**: Element Plus / Naive UI
- **CSS**: TailwindCSS / UnoCSS
- **后端**: Node.js + Express + MySQL/PostgreSQL
- **构建**: Vite / Webpack / HBuilderX
- **包管理**: pnpm / npm / yarn

## 快速开始

1. 将 `fe-skill` 目录放到你的项目中或 `~/.claude/` 下
2. Claude Code 会自动识别 `.claude-plugin/plugin.json`
3. 使用 `/plan` 开始规划你的功能开发

## 致谢

本项目的设计理念和工作流参考自：
- [everything-claude-code](https://github.com/affaan-m/everything-claude-code) by Affaan Mustafa
