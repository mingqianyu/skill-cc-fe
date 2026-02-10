<!--
  【设计理念】
  参考开源项目的 update-docs.md。
  核心思想：文档的"单一事实来源"是代码本身。
  package.json 和 .env.example 是文档的源头。
-->

# /update-docs 命令

从代码中同步更新项目文档。

## 执行流程

1. 读取 `package.json` scripts，生成脚本参考表
2. 读取 `.env.example`，提取环境变量文档
3. 识别 90 天未更新的过期文档
4. 展示变更差异摘要

## 单一事实来源

- `package.json` → 脚本命令文档
- `.env.example` → 环境变量文档
