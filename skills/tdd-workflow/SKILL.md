---
name: tdd-workflow
description: TDD 工作流技能 - RED/GREEN/REFACTOR 循环的详细指导
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 tdd-workflow skill。

  【TDD Skill vs TDD Command vs TDD Agent 的关系】
  这三者是开源项目"关注点分离"的典型体现：
  - tdd-workflow (skill): 深度知识库，包含 TDD 的理论和示例
  - tdd (command): 用户入口，定义 /tdd 命令的行为
  - tdd-guide (agent): 执行者，实际执行 TDD 流程的专家角色

  三者协作：
  用户输入 /tdd → command 定义流程 → 调用 agent 执行 → agent 参考 skill 中的知识
-->

# TDD 工作流技能

## RED 阶段：写失败测试

```typescript
// 1. 先定义接口
interface CalculateDiscountParams {
  originalPrice: number
  discountRate: number
  memberLevel: 'normal' | 'vip' | 'svip'
}

// 2. 写测试（此时函数还不存在）
describe('calculateDiscount', () => {
  it('普通用户应用折扣率', () => {
    const result = calculateDiscount({
      originalPrice: 100,
      discountRate: 0.8,
      memberLevel: 'normal'
    })
    expect(result).toBe(80)
  })

  it('VIP 用户额外 5% 折扣', () => {
    const result = calculateDiscount({
      originalPrice: 100,
      discountRate: 0.8,
      memberLevel: 'vip'
    })
    expect(result).toBe(76) // 80 * 0.95
  })

  it('价格不能为负数', () => {
    const result = calculateDiscount({
      originalPrice: -100,
      discountRate: 0.8,
      memberLevel: 'normal'
    })
    expect(result).toBe(0)
  })
})
```

## GREEN 阶段：最小实现

```typescript
function calculateDiscount(params: CalculateDiscountParams): number {
  const { originalPrice, discountRate, memberLevel } = params

  if (originalPrice <= 0) return 0

  let price = originalPrice * discountRate

  if (memberLevel === 'vip') price *= 0.95
  if (memberLevel === 'svip') price *= 0.9

  return Math.round(price * 100) / 100
}
```

## REFACTOR 阶段：改善代码

在测试全部通过的保护下，安全地重构。
重构后必须再次运行测试确认通过。
