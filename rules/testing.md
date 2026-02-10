<!--
  【设计理念】
  测试规范。参考开源项目的 testing.md。

  【开源项目的测试哲学】
  1. TDD 是强制流程，不是可选项
  2. 最低 80% 覆盖率是底线
  3. 三类测试都需要：单元、集成、E2E
  4. 测试失败时优先修实现，而非修测试

  【为什么 TDD 对 AI 编程特别重要？】
  Claude 生成的代码可能有隐藏 Bug。
  先写测试再实现，可以：
  - 明确需求边界（测试即文档）
  - 自动验证实现是否正确
  - 重构时有安全网
-->

# 测试规范

## 最低覆盖率：80%

三类测试均需要：

1. **单元测试** - 函数、工具函数、组合式函数
2. **集成测试** - API 接口、数据库操作
3. **E2E 测试** - 关键用户流程

## TDD 强制流程

```
RED → GREEN → REFACTOR → REPEAT

RED:      先写一个会失败的测试
GREEN:    写最少的代码让测试通过
REFACTOR: 在测试保护下重构代码
REPEAT:   下一个场景
```

### 具体步骤

1. 先写测试（RED）
2. 运行测试 - 确认失败
3. 写最小实现（GREEN）
4. 运行测试 - 确认通过
5. 重构优化（REFACTOR）
6. 验证覆盖率（≥ 80%）

## Vue 组件测试示例

```typescript
// components/__tests__/UserCard.test.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import UserCard from '../UserCard.vue'

describe('UserCard', () => {
  it('应该显示用户名称', () => {
    const wrapper = mount(UserCard, {
      props: { user: { id: '1', name: '张三' } }
    })
    expect(wrapper.text()).toContain('张三')
  })

  it('点击时应触发 select 事件', async () => {
    const wrapper = mount(UserCard, {
      props: { user: { id: '1', name: '张三' } }
    })
    await wrapper.trigger('click')
    expect(wrapper.emitted('select')).toBeTruthy()
  })
})
```

## 测试失败排查

1. 使用 **tdd-guide** 代理
2. 检查测试隔离性
3. 验证 mock 是否正确
4. **优先修实现，而非修测试**（除非测试本身写错）

## 代理支持

- **tdd-guide** - 新功能时主动使用，强制先写测试
