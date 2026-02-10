<!--
  【设计理念】
  CLAUDE.md 是放在项目根目录的配置文件。
  它是 Claude Code 读取项目上下文的入口。

  【CLAUDE.md vs plugin.json 的区别】
  - plugin.json: 定义插件结构（skills/agents 路径）
  - CLAUDE.md: 定义项目规范（编码风格、架构、命令）

  【为什么需要 CLAUDE.md？】
  每个项目都有自己的约定：
  - 目录结构不同
  - 技术栈不同
  - 编码风格不同
  CLAUDE.md 让 Claude 快速理解"这个项目的规矩"。

  【本文件是示例模板】
  复制到你的项目根目录，根据实际情况修改。
-->

# 项目 CLAUDE.md 示例

> 将此文件复制到项目根目录并根据实际情况修改。

## 项目概述

[简述项目功能、技术栈]

## 关键规则

### 1. 代码组织
- 多小文件优于少大文件（200-400 行为宜，最多 800 行）
- 高内聚、低耦合
- 按功能/领域组织目录

### 2. 代码风格
- 使用 `<script setup lang="ts">` 编写 Vue 组件
- 不可变数据模式（禁止直接修改对象/数组）
- 生产代码禁止 console.log
- 完整的错误处理（try/catch）

### 3. 测试
- TDD：先写测试再实现
- 最低 80% 覆盖率
- 单元测试 + 集成测试

### 4. 安全
- 禁止硬编码密钥
- 环境变量管理敏感数据
- 参数化查询防 SQL 注入
- 校验所有用户输入

## 目录结构

```
src/
├── views/            # 页面组件
├── components/       # 公共组件
├── composables/      # 组合式函数
├── stores/           # Pinia 状态管理
├── api/              # API 接口定义
├── utils/            # 工具函数
├── types/            # TypeScript 类型
└── router/           # 路由配置
```

## 环境变量

```bash
# 必需
VITE_API_BASE_URL=
VITE_APP_TITLE=

# 可选
VITE_DEBUG=false
```

## 可用命令

- `/plan` - 创建实施计划
- `/tdd` - 测试驱动开发
- `/code-review` - 代码审查
- `/verify` - 综合验证
- `/build-fix` - 修复构建错误

## Git 规范

- 约定式提交: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- 禁止直接提交到 main 分支
- PR 需要审查通过
- 所有测试必须通过
