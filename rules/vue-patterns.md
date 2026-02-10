<!--
  【设计理念】
  这是 Vue 3 专属的开发模式规则。
  开源项目中 frontend-patterns 是通用前端模式（React 为主），
  这里我们将其适配为 Vue 3 的最佳实践。

  【为什么需要 Vue 专属规则？】
  Vue 3 有自己独特的设计哲学：
  1. Composition API 替代 Options API
  2. 响应式系统基于 Proxy
  3. 组合式函数（composables）替代 mixins
  4. <script setup> 语法糖简化组件定义
  Claude 需要明确知道这些约定，才能生成符合规范的代码。
-->

# Vue 3 开发模式规则

## 组件编写规范

### 使用 `<script setup>` 语法

<!--
  <script setup> 是 Vue 3.2+ 推荐的组件编写方式：
  - 更简洁：无需 return 暴露变量
  - 更好的 TypeScript 支持
  - 更好的 IDE 推断
  - 编译时优化，性能更好
-->

```vue
<!-- ✅ 推荐：<script setup> -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { UserInfo } from '@/types/user'

// Props 定义（使用 TypeScript 类型）
const props = defineProps<{
  userId: string
  showAvatar?: boolean
}>()

// Emits 定义
const emit = defineEmits<{
  update: [value: string]
  delete: [id: string]
}>()

// 响应式状态
const loading = ref(false)
const userInfo = ref<UserInfo | null>(null)

// 计算属性
const displayName = computed(() => {
  return userInfo.value?.nickname || userInfo.value?.username || '未知用户'
})
</script>

<template>
  <div class="user-card">
    <span>{{ displayName }}</span>
  </div>
</template>
```

### 组件结构顺序

```vue
<script setup lang="ts">
// 1. 导入语句
import { ref, computed, onMounted } from 'vue'

// 2. Props 和 Emits 定义
const props = defineProps<{ /* ... */ }>()
const emit = defineEmits<{ /* ... */ }>()

// 3. 组合式函数调用
const { data, loading } = useUserList()

// 4. 响应式状态（ref / reactive）
const searchKeyword = ref('')

// 5. 计算属性
const filteredList = computed(() => { /* ... */ })

// 6. 方法
function handleSearch() { /* ... */ }

// 7. 生命周期钩子
onMounted(() => { /* ... */ })
</script>

<template>
  <!-- 模板内容 -->
</template>

<style scoped>
/* 样式（始终使用 scoped） */
</style>
```

## 组合式函数（Composables）

<!--
  【设计理念】组合式函数是 Vue 3 最重要的代码复用方式
  开源项目强调"Custom Hooks Pattern"，在 Vue 中对应 composables。
  核心原则：
  1. 以 use 开头命名
  2. 封装可复用的有状态逻辑
  3. 返回响应式数据和方法
  4. 一个 composable 只做一件事
-->

```typescript
// ✅ 好的 composable：职责单一、命名清晰
// composables/usePageList.ts
import { ref, reactive } from 'vue'

interface UsePageListOptions<T> {
  fetchApi: (params: PageParams) => Promise<PageResult<T>>
  defaultPageSize?: number
}

export function usePageList<T>(options: UsePageListOptions<T>) {
  const { fetchApi, defaultPageSize = 10 } = options

  const loading = ref(false)
  const list = ref<T[]>([]) as Ref<T[]>
  const pagination = reactive({
    current: 1,
    pageSize: defaultPageSize,
    total: 0
  })

  async function fetchList() {
    loading.value = true
    try {
      const result = await fetchApi({
        page: pagination.current,
        pageSize: pagination.pageSize
      })
      list.value = result.list
      pagination.total = result.total
    } finally {
      loading.value = false
    }
  }

  function handlePageChange(page: number) {
    pagination.current = page
    fetchList()
  }

  return {
    loading,
    list,
    pagination,
    fetchList,
    handlePageChange
  }
}
```

## Pinia 状态管理

<!--
  【为什么用 Pinia 而不是 Vuex？】
  Pinia 是 Vue 官方推荐的状态管理方案：
  - 完整的 TypeScript 支持
  - 无 mutations，只有 state/getters/actions
  - 支持 Composition API 风格
  - 更轻量、更简单
-->

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { UserInfo } from '@/types/user'
import { getUserInfo, login } from '@/api/user'

// ✅ 推荐：使用 Setup Store 风格（Composition API）
export const useUserStore = defineStore('user', () => {
  // state
  const token = ref<string>('')
  const userInfo = ref<UserInfo | null>(null)

  // getters
  const isLoggedIn = computed(() => !!token.value)
  const userName = computed(() => userInfo.value?.name ?? '')

  // actions
  async function loginAction(username: string, password: string) {
    const res = await login({ username, password })
    token.value = res.token
  }

  async function fetchUserInfo() {
    if (!token.value) return
    userInfo.value = await getUserInfo()
  }

  function logout() {
    token.value = ''
    userInfo.value = null
  }

  return {
    token,
    userInfo,
    isLoggedIn,
    userName,
    loginAction,
    fetchUserInfo,
    logout
  }
})
```

## 模板规范

```vue
<template>
  <!-- ✅ 使用语义化标签 -->
  <main class="page-container">
    <!-- ✅ v-if / v-else 紧邻使用 -->
    <el-skeleton v-if="loading" :rows="5" animated />
    <div v-else-if="error" class="error-state">
      {{ error.message }}
    </div>
    <div v-else class="content">
      <!-- ✅ 列表渲染必须有 key -->
      <UserCard
        v-for="user in userList"
        :key="user.id"
        :user="user"
        @click="handleUserClick(user)"
      />
      <!-- ✅ 空状态处理 -->
      <el-empty v-if="userList.length === 0" description="暂无数据" />
    </div>
  </main>
</template>
```

## 禁止事项

- ❌ 禁止使用 Options API（统一使用 Composition API）
- ❌ 禁止使用 mixins（使用 composables 替代）
- ❌ 禁止在模板中写复杂逻辑（提取为 computed 或方法）
- ❌ 禁止 v-for 和 v-if 同时用在同一元素上
- ❌ 禁止直接修改 props
- ❌ 禁止使用 `any` 类型（使用具体类型或 `unknown`）
