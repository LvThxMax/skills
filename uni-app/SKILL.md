---
name: uni-app
description: uni-app 跨端开发指南，覆盖微信、支付宝、抖音等小程序与 App。在用户编写 uni-app 页面、组件、跨端适配或调用 uni API 时使用
---

# uni-app 编程指南

## 核心原则

1. **简洁** — 页面逻辑保持扁平；共用逻辑抽 composable 或组件，不为单平台写死分支
2. **性能好** — 控制包体积与渲染开销；图片压缩与懒加载；减少 setData 数据量与频率；列表分页或虚拟列表
3. **扩展性好** — 平台差异用条件编译隔离；业务与 UI 分离；API 层统一封装，便于增删平台
4. **不用 BEM** — CSS 类名扁平、语义化（如 `order-item`、`is-selected`），禁止 `block__element--modifier`

## 技术约定

- Vue 3 + `<script setup>`（与项目配置一致）
- 页面放 `pages/`，复用组件放 `components/`
- 优先 `uni.*` API，不混用各平台私有 API（除非条件编译块内）

## 跨端适配

```vue
<!-- #ifdef MP-WEIXIN -->
<!-- 微信特有 -->
<!-- #endif -->

<!-- #ifdef MP-ALIPAY -->
<!-- 支付宝特有 -->
<!-- #endif -->
```

- 样式用 `rpx` 做适配；安全区用 `env(safe-area-inset-*)`
- 平台差异集中在适配层，业务代码保持统一
- 新增平台时只扩展条件编译与适配模块，不改核心业务流程

## 页面与组件

- 单页职责清晰：数据获取 → 状态 → 展示
- 列表项抽成独立组件，避免模板重复
- 事件、props 类型明确；跨页通信用 Pinia 或 `uni.$emit`（按项目约定）

## 性能要点

- 图片：合适尺寸、webp（平台支持时）、`lazy-load`
- 列表：分页加载；长列表用 `recycle-view` 或官方虚拟列表方案
- 减少 WXML/视图节点层级与数量
- 避免在 `onShow` / 滚动回调中频繁请求或大量赋值
- 分包加载：主包放核心页，非首屏功能放分包

## 扩展要点

- 请求封装在 `api/` 或 `utils/request`，统一 baseURL、拦截器、错误处理
- 常量、枚举、平台配置集中管理
- 可复用业务能力（登录、支付、分享）封装为 composable 或 service

## 样式

- 类名语义化、扁平化；状态类用 `is-` / `has-` 前缀
- 优先复用项目全局样式与变量
- 各端表现不一致时，用条件编译样式而非大量 `!important`

## 禁止

- BEM 类名（`card__title--active` 等）
- 在业务代码里散落 `// #ifdef`（应下沉到适配层）
- 主包塞入大图、无用依赖导致体积膨胀
- 未经要求的原生插件或平台私有 SDK
- 用 index 作 `v-for` 的 key
