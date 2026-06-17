---
name: vue3
description: Vue 3 前端开发指南，涵盖 Composition API、Pinia、Vue Router、Vite。在用户编写或修改 Vue 3 页面、组件、状态、路由等前端代码时使用。
---

# Vue 3 编程指南

## 核心原则

1. **简洁** — 只写必要代码；逻辑复杂时再拆 composable 或子组件，不为简单场景造抽象
2. **性能好** — 路由与重型组件懒加载；列表用稳定 `:key`；派生用 `computed`，少滥用 `watch`；大列表考虑虚拟滚动
3. **扩展性好** — 按职责分层（`views` / `components` / `composables` / `stores` / `api`）；跨组件逻辑进 composable，共享状态进 Pinia
4. **不用 BEM** — CSS 类名扁平、语义化（如 `user-card`、`is-active`），禁止 `block__element--modifier`

## 技术约定

- `<script setup lang="ts">` + Composition API
- Pinia 管状态，Vue Router 4 管路由，Vite 构建
- 复用项目已有 HTTP 封装与目录结构，不重复造轮子

## 组件

```vue
<script setup lang="ts">
// imports → props/emits → composables/store → state/computed → methods/watchers → lifecycle
</script>

<template>...</template>

<style scoped lang="scss">...</style>
```

- Props / Emits 必须类型化；`v-model` 优先 `defineModel()`
- 单向数据流，子组件不改 props
- 单文件过长时拆 composable 或子组件，而非堆逻辑

## 样式

- 默认 `scoped`；穿透用 `:deep()`
- 类名语义化、扁平化；状态类用 `is-` / `has-` 前缀
- 颜色、间距用 CSS 变量 / 设计 token，避免内联 style

## 性能要点

- 页面：`() => import('@/views/Xxx.vue')`
- `v-for` 用唯一 id 作 key，不用 index
- 频繁显隐用 `v-show`，条件极少满足用 `v-if`
- 模板内不写复杂表达式，过滤 / 排序放 `computed`
- 副作用在 `onUnmounted` 或 composable 内清理

## 扩展要点

- Store 按业务域拆分，只放跨页面共享状态
- Composable 单一职责，命名 `useXxx`
- API 请求与类型定义独立于组件，便于复用与测试

## 禁止

- BEM 类名（`card__title--active` 等）
- Vue 2 写法（`this`、`filters`、`.sync`）
- 无关依赖、全局状态、目录结构调整（除非用户要求）
- 为简单 UI 写多余 wrapper 或工具层
