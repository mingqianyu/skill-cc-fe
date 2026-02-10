<!--
  【设计理念】
  /orchestrate 是开源项目最高级的命令。
  它实现了"多代理编排"——按顺序调用多个代理完成复杂任务。

  【核心概念：Handoff（交接）】
  代理之间通过"交接文档"传递上下文。
  每个代理完成后，生成一份结构化的交接文档，
  包含：做了什么、发现了什么、建议下一步做什么。
  下一个代理读取交接文档后继续工作。

  【为什么需要编排？】
  单个代理的能力有限，复杂任务需要多个专家协作。
  编排 = 定义专家的协作顺序和信息传递方式。
-->

# /orchestrate 命令

多代理顺序工作流，用于复杂任务。

## 用法

`/orchestrate [工作流类型] [任务描述]`

## 工作流类型

### feature（新功能）
```
planner → tdd-guide → code-reviewer → security-reviewer
```

### bugfix（修复 Bug）
```
探索问题 → tdd-guide → code-reviewer
```

### refactor（重构）
```
architect → code-reviewer → tdd-guide
```

### security（安全审查）
```
security-reviewer → code-reviewer → architect
```

## 交接文档格式

代理之间的信息传递：

```markdown
## 交接: [上一个代理] → [下一个代理]

### 上下文
[做了什么的摘要]

### 发现
[关键发现或决策]

### 修改的文件
[文件列表]

### 待解决问题
[未解决的事项]

### 建议
[下一步建议]
```

## 最终报告格式

```
编排报告
========
工作流: feature
任务: [描述]
代理链: planner → tdd-guide → code-reviewer

摘要: [一段话总结]
文件变更: [列表]
测试结果: [通过/失败]
建议: [发布 / 需要修改 / 阻塞]
```
