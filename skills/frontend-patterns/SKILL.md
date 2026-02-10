---
name: frontend-patterns
description: 前端通用模式 - 性能优化、错误边界、表单处理、可访问性
version: 1.0.0
---

<!--
  【设计理念】
  参考开源项目的 frontend-patterns skill。
  开源项目以 React 为主，这里适配为 Vue 3 生态。

  【与 vue3-patterns 的区别】
  - vue3-patterns: Vue 框架特有的模式（composables、Pinia 等）
  - frontend-patterns: 通用前端模式（性能、表单、错误处理等）
-->

# 前端通用模式

## 性能优化模式

### 虚拟滚动（大列表）

```vue
<!-- 使用 @vueuse/core 的虚拟列表 -->
<script setup lang="ts">
import { useVirtualList } from '@vueuse/core'

const { list, containerProps, wrapperProps } = useVirtualList(
  largeList,  // 大数据列表
  { itemHeight: 60 }
)
</script>

<template>
  <div v-bind="containerProps" style="height: 400px; overflow-y: auto">
    <div v-bind="wrapperProps">
      <div v-for="item in list" :key="item.index" style="height: 60px">
        {{ item.data.name }}
      </div>
    </div>
  </div>
</template>
```

### 组件懒加载

```typescript
import { defineAsyncComponent } from 'vue'

// 路由级懒加载
const routes = [
  {
    path: '/dashboard',
    component: () => import('@/views/Dashboard.vue')
  }
]

// 组件级懒加载
const HeavyChart = defineAsyncComponent(
  () => import('@/components/HeavyChart.vue')
)
```

## 表单处理模式

```vue
<script setup lang="ts">
import { reactive } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'

const formRef = ref<FormInstance>()

const form = reactive({
  username: '',
  email: '',
  password: ''
})

const rules: FormRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 2, max: 20, message: '长度 2-20 个字符', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '邮箱格式不正确', trigger: 'blur' }
  ]
}

async function handleSubmit() {
  const valid = await formRef.value?.validate().catch(() => false)
  if (!valid) return
  // 提交逻辑
}
</script>
```

## 错误处理模式

```typescript
// 全局错误处理
app.config.errorHandler = (err, instance, info) => {
  console.error('全局错误:', err, info)
  // 上报错误监控
}
```
