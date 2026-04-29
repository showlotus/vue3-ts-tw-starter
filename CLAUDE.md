# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

| 命令               | 说明                              |
| ------------------ | --------------------------------- |
| `pnpm dev`         | 启动 Vite 开发服务器（端口 5173） |
| `pnpm build`       | 生产构建                          |
| `pnpm preview`     | 构建后预览生产版本                |
| `pnpm test`        | 运行 Vitest 单元/组件测试         |
| `pnpm coverage`    | 运行 Vitest 并生成覆盖率报告      |
| `pnpm lint`        | ESLint 检查并自动修复             |
| `pnpm lint:oxlint` | Oxlint 检查并自动修复             |
| `pnpm ts`          | vue-tsc 类型检查                  |

运行单个测试文件：`pnpm vitest run path/to/test.spec.ts`

## 项目架构

这是一个 **Vite + Vue 3 + TypeScript + Tailwind CSS v4** 的启动模板。包管理器为 **pnpm**（v10），使用 ES 模块。

### 核心启动流程

[main.ts](src/main.ts) 是应用入口，按顺序挂载插件：Pinia → Router → Unhead → 挂载 `#app`。Pinia 初始化时会将 router 实例注入 store（`store.router`），使 store 内可直接进行路由跳转。

### 路径别名

`@` 映射到 `src/` 目录，在 vite.config.ts 和 tsconfig 中均已配置。

### 路由与页面

路由定义在 [router.ts](src/router.ts)，使用 `createWebHistory`。页面组件放 `src/pages/`，组件放 `src/components/`。路由 meta 的 `title` 字段已通过 [interface-extensions.d.ts](interface-extensions.d.ts) 扩展类型。

### Store 设计

Pinia store 定义在 [store.ts](src/store.ts)，名为 `main`。包含：

- `debug`：开发模式标识
- `appMeta`：版本号和构建时间（从构建时的 ENV 注入）
- `isInitialized` / `count` 等示例状态
- 支持 HMR 热更新（`acceptHMRUpdate`）

### Head 元标签管理

使用 `@unhead/vue`，在 [App.vue](src/App.vue) 中通过 `useHead` 设置响应式标题和 OG 标签，标题从路由 meta 动态读取。

### 类型声明文件

- [env.d.ts](env.d.ts)：为 `import.meta.env` 扩展 `VITE_APP_VERSION` 和 `VITE_APP_BUILD_EPOCH` 类型
- [interface-extensions.d.ts](interface-extensions.d.ts)：扩展 `vue-router` 的 `RouteMeta`（增加 `title`）和 `pinia` 的 `PiniaCustomProperties`（增加 `router`）

### CSS / Tailwind

Tailwind CSS v4 通过 `@tailwindcss/vite` 插件集成（非 PostCSS 方式）。基础样式在 [base.css](src/assets/base.css) 中，使用 `@import 'tailwindcss'` 和 `@plugin` 指令加载 typography 和 iconify 插件。图标使用 Iconify Tailwind 格式：`<span class="iconify mdi--icon-name"></span>`。

### 测试架构

- **Vitest**：配置在 [vitest.config.ts](vitest.config.ts)，继承 vite.config.ts，使用 jsdom 环境。测试文件放在 `src/**/*.spec.ts` 和 `tests/unit/**/*.test.ts`。全局安装了 `@pinia/testing`（stubActions: false），并设置了 `global.runningTests = true`。

### 环境变量

`VITE_APP_VERSION` 和 `VITE_APP_BUILD_EPOCH` 在构建时由 vite.config.ts 从 package.json 版本号和当前时间戳自动注入，不在 `.env` 文件中定义。
