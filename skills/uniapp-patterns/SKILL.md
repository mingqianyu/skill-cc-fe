---
name: uniapp-patterns
description: uni-app 跨端开发模式、条件编译、多端适配最佳实践
version: 1.0.0
---

<!--
  【设计理念】
  uni-app 是国内主流的跨端开发框架。
  这个 skill 是 fe-skill 独有的，开源项目中没有。

  【为什么需要 uni-app 专属技能？】
  uni-app 有很多平台特有的约束：
  1. 条件编译（#ifdef）是核心机制
  2. 不同平台的 API 差异
  3. 小程序的生命周期与 Vue 不同
  4. 样式单位（rpx vs px）
  Claude 需要了解这些差异才能生成正确的跨端代码。
-->

# uni-app 跨端开发模式

## 条件编译

```vue
<template>
  <!-- #ifdef MP-WEIXIN -->
  <button open-type="getUserInfo">微信登录</button>
  <!-- #endif -->

  <!-- #ifdef H5 -->
  <button @click="h5Login">账号登录</button>
  <!-- #endif -->

  <!-- #ifdef APP-PLUS -->
  <button @click="appLogin">一键登录</button>
  <!-- #endif -->
</template>

<script setup lang="ts">
// #ifdef MP-WEIXIN
import { wechatLogin } from '@/utils/wechat'
// #endif
</script>

<style>
/* #ifdef MP-WEIXIN */
.container { padding: 20rpx; }
/* #endif */

/* #ifdef H5 */
.container { padding: 10px; }
/* #endif */
</style>
```

## 生命周期

```typescript
// 页面生命周期（uni-app 特有）
import { onLoad, onShow, onHide, onReachBottom, onPullDownRefresh } from '@dcloudio/uni-app'

onLoad((options) => {
  // 页面加载，接收路由参数
  const { id } = options ?? {}
})

onShow(() => {
  // 页面显示（每次进入都触发）
})

onReachBottom(() => {
  // 触底加载更多
  loadMore()
})

onPullDownRefresh(() => {
  // 下拉刷新
  refresh().finally(() => {
    uni.stopPullDownRefresh()
  })
})
```

## 跨端适配原则

1. **样式单位**: 小程序用 `rpx`，H5 用 `px` 或 `rem`
2. **存储 API**: 统一用 `uni.setStorageSync` / `uni.getStorageSync`
3. **路由跳转**: 统一用 `uni.navigateTo` / `uni.switchTab`
4. **网络请求**: 统一用 `uni.request` 或封装的 axios 适配器
5. **平台差异**: 用条件编译处理，不要用运行时判断
