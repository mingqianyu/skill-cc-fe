<!--
  【设计理念】
  /checkpoint 是开源项目的"存档点"概念。
  类似游戏存档，在关键节点保存状态，
  方便后续对比和回滚。
-->

# /checkpoint 命令

创建或验证工作流检查点。

## 用法

`/checkpoint [create|verify|list] [名称]`

## 创建检查点

1. 运行 `/verify quick` 确保当前状态干净
2. 创建 git stash 或 commit
3. 记录到 `.claude/checkpoints.log`
4. 报告检查点已创建

## 验证检查点

对比当前状态与检查点：
- 新增/修改的文件
- 测试通过率变化
- 覆盖率变化

## 典型流程

```
开始 → /checkpoint create "feature-start"
  ↓
实现 → /checkpoint create "core-done"
  ↓
测试 → /checkpoint verify "core-done"
  ↓
重构 → /checkpoint create "refactor-done"
  ↓
提PR → /checkpoint verify "feature-start"
```
