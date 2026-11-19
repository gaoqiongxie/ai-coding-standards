# 前端编码规范与安全基线（AI 协作版）

> 面向 AI 编程工具的 Vue 2/3 通用前端规范（示例以 Vue 2 + axios 项目为背景，原则同样适用于 React 等其他栈）。
> 规则仅对新开发代码生效，历史代码保持不动。

---

## 1. API 调用铁律

> **AI 为什么爱犯**：AI 生成页面时习惯在组件里直接写 `axios.get(...)`，绕过拦截器，导致签名、token、401 处理、错误提示全部失效。

- 页面组件**禁止直接写 axios 请求**，必须经项目的 API 层调用（如 `this.$api.xxx()` / `import { getCustomerList } from '@/apis/customer'`）
- API 层职责：baseURL、token、签名、401 引导重新登录、统一错误 Toast——全部收敛在 axios 拦截器与 `apis/` 目录
- API 按模块拆文件聚合在 `src/apis/`；PC/移动端统一走同一套 API，不分别维护数据源

```javascript
// ✅ 正确
this.$api.getCustomerList({ page: 1, pageSize: 10 }).then(res => {
  this.list = res.data
})

// ❌ 禁止
axios.get('/api/customer/list').then(...)
```

---

## 2. 组件与目录规范

- Vue SFC 顺序：`<template>` → `<script>` → `<style scoped>`
- 遵循项目现有代码风格，先看同模块已有文件的写法；复用逻辑优先用项目现有模式（Mixin/composable）
- 组件文件 PascalCase 多单词（`KpiCard.vue`），模板引用 kebab-case（`<kpi-card />`）
- 页面组件放 `src/views/<模块>/`，公共组件放 `src/components/`，工具函数放 `src/utils/`
- 金额计算用 mathjs 等精度库，**禁止原生浮点运算**；时间处理用 moment/dayjs
- 适配方案（px2rem、双端断点等）按项目现有机制执行，注意第三方 UI 库是否排除转换

---

## 3. 模板规范

- **`v-for` 必须加 `:key`**，key 在循环中唯一；**禁止同一元素上 `v-if` + `v-for` 并存**（把 `v-if` 上移或用 computed 先过滤）
- 属性顺序约定：原生属性（class/ref/id）→ `v-for`/`:key` → `v-if`/`v-show` → `v-model` → `v-bind`(:) → `v-on`(@)
- 可点击元素用 `<button>` / `<a>`，不用 `<div>` 冒充；icon-only 按钮加 `aria-label`；表单项必须有 `<label>` 或等效描述
- 外链加 `rel="noopener noreferrer"`

---

## 4. 安全基线（红线）

| 红线 | 说明 |
|------|------|
| **默认禁止 `v-html`** | 必须使用时要先做白名单净化（如 DOMPurify），输入一律视为不可信 |
| **禁止 `this.$parent`** | 跨层访问破坏组件封装，用 props/emit 或状态管理 |
| **禁止提交 `console.log` / `debugger`** | 提交前清理；合并前 ESLint 检查通过 |

---

## 5. 错误处理与状态

- 网络/数据异常不得导致页面崩溃
- 列表失败：Toast 提示 + 空数据渲染；详情失败：Toast 提示 + 空态
- 统一用项目 UI 库的 Toast 组件提示，**禁止静默失败**
- 页面负责 UI 组合、路由交互、调 API、处理 loading/空态/错误态，业务逻辑下沉到 API 层/store

---

## 6. 工程与提交

- 路由页面懒加载：`() => import('@/views/XxxPage.vue')`
- 大列表评估虚拟滚动或分页；关注构建 chunk 警告，必要时按模块拆包
- import 顺序：第三方库 → 本地绝对路径（`@/...`）→ 相对路径，同组内按字母排序
- 合并前必须 `npm run build` 通过，`npm run lint` 无新增告警
- 多环境代理前缀（如 `/dev-api`、`/test-api`、`/api`）按项目 `vue.config.js`/构建配置约定，开发环境用 URL 参数切换代理目标
