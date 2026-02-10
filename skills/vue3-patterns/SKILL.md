---
name: vue3-patterns
description: Vue 3 组件模式、Composition API、响应式系统最佳实践
version: 1.0.0
---

<!--
  【设计理念】
  Skills 与 Rules 的核心区别：
  - Rules = 始终加载的强制规则（轻量、简洁）
  - Skills = 按需引用的深度知识（详细、有示例）

  Rules 告诉 Claude "必须做什么"，
  Skills 告诉 Claude "怎么做得好"。

  【SKILL.md 文件格式】
  每个 skill 是一个目录，包含一个 SKILL.md 文件。
  frontmatter 中的 name 和 description 用于 Claude 判断何时引用此技能。
  当用户的任务与 description 匹配时，Claude 会自动加载此技能。
-->

# Vue 3 组件模式

## 组合式函数模式（Composables）

<!--
  组合式函数是 Vue 3 最重要的代码复用机制。
  它替代了 Vue 2 的 mixins，解决了：
  1. 命名冲突问题
  2. 数据来源不清晰问题
  3. 类型推断困难问题
-->

### 基础模式：封装异步数据获取

```typescript
// composables/useFetch.ts
import { ref, shallowRef } from 'vue'

export function useFetch<T>(url: string | Ref<string>) {
  const data = shallowRef<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      const response = await fetch(unref(url))
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e : new Error(String(e))
    } finally {
      loading.value = false
    }
  }

  // 如果 url 是响应式的，自动监听变化
  if (isRef(url)) {
    watch(url, execute, { immediate: true })
  } else {
    execute()
  }

  return { data, error, loading, execute }
}
```

### 进阶模式：带分页的列表

```typescript
// composables/usePageList.ts
import { ref, reactive, type Ref } from 'vue'

interface PageParams {
  page: number
  pageSize: number
  [key: string]: any
}

interface PageResult<T> {
  list: T[]
  total: number
}

interface UsePageListOptions<T> {
  // 获取数据的 API 函数
  fetchApi: (params: PageParams) => Promise<PageResult<T>>
  // 默认每页条数
  defaultPageSize?: number
  // 是否立即加载
  immediate?: boolean
}

export function usePageList<T>(options: UsePageListOptions<T>) {
  const { fetchApi, defaultPageSize = 10, immediate = true } = options

  const loading = ref(false)
  const list = ref<T[]>([]) as Ref<T[]>
  const pagination = reactive({
    current: 1,
    pageSize: defaultPageSize,
    total: 0
  })

  // 额外的查询参数
  const queryParams = ref<Record<string, any>>({})

  async function fetchList() {
    loading.value = true
    try {
      const result = await fetchApi({
        page: pagination.current,
        pageSize: pagination.pageSize,
        ...queryParams.value
      })
      list.value = result.list
      pagination.total = result.total
    } catch (error) {
      console.error('获取列表失败:', error)
      list.value = []
    } finally {
      loading.value = false
    }
  }

  // 翻页
  function handlePageChange(page: number) {
    pagination.current = page
    fetchList()
  }

  // 搜索（重置到第一页）
  function handleSearch(params: Record<string, any>) {
    queryParams.value = params
    pagination.current = 1
    fetchList()
  }

  // 刷新当前页
  function refresh() {
    fetchList()
  }

  if (immediate) {
    fetchList()
  }

  return {
    loading,
    list,
    pagination,
    fetchList,
    handlePageChange,
    handleSearch,
    refresh
  }
}
```
