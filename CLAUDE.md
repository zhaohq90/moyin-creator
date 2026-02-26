# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

**语言约定**：项目文档和代码注释优先使用中文。

## 常用命令

```bash
npm run dev          # 以开发模式启动 Electron 应用
npm run build        # 构建 Windows 安装包（electron-vite build + electron-builder）
npm run lint         # ESLint 检查（零警告容忍）
npm run preview      # 预览生产构建
```

本项目未配置测试框架。

## 架构概览

**魔因漫创（Moyin Creator）** 是一款基于 Electron 的桌面应用，用于 AI 驱动的动漫/短剧分镜制作。核心工作流：剧本 → 角色 → 场景 → 导演 → S级（视频生成）。

### 分层结构

**Electron 层** (`/electron/`)
- `main.ts` — IPC 处理、文件系统、本地存储、窗口管理
- `preload.ts` — 安全桥接层，向渲染进程暴露 `window.electronAPI`

**状态管理层** (`/src/stores/` — 17 个 Zustand store)
- 所有 store 使用带项目作用域路由的持久化中间件
- 项目专属数据：`_p/{projectId}/{storeName}.json`
- 共享资源：`_shared/{storeName}.json`
- 核心 store：`director-store.ts`（最大，约 71KB）、`script-store.ts`、`character-library-store.ts`、`scene-store.ts`、`api-config-store.ts`
- 角色/场景/媒体资源可在项目作用域与共享存储之间切换

**AI 核心包** (`/src/packages/ai-core/`)
- 内部包别名为 `@opencut/ai-core`
- 基于 Mustache 风格模板的提示词编译
- 角色圣经管理器（6 层身份锚定，保障角色一致性）
- 任务轮询、队列管理、API Key 轮换与负载均衡
- 支持多 AI 服务商的抽象层

**UI 层** (`/src/components/`)
- 标签页导航：剧本、角色、场景、导演、S级、媒体、导出、设置
- 基于 `react-resizable-panels` 的可调整面板布局
- `panels/` — 主功能面板；`ui/` — 65 个可复用基础组件（基于 Radix UI）

**服务/工具层** (`/src/lib/`)
- `ai/` — AI 服务集成
- `storyboard/` — 分镜生成流水线
- `character/` — 角色提示词服务
- `freedom/` — 自由创作模式工具
- `constants/` — 视觉风格、摄影参数配置

### 关键模式

**存储路由**：各 store 使用自定义适配器，通过 Electron IPC 将读写操作路由到文件系统。`project-store.ts` 管理当前激活的项目上下文，其他 store 根据当前项目 ID 推导各自的存储路径。

**AI 任务流**：请求经由 `ai-core/api/` 中的队列处理，具备重试逻辑和 Key 轮换机制。提示词在发送前会结合角色圣经数据从模板编译生成。

**类型别名**：`@/*` → `./src/*`，`@opencut/ai-core` → `./src/packages/ai-core`

**样式**：Tailwind CSS v4（PostCSS 插件，无 `tailwind.config.js`）。组件变体使用 `class-variance-authority`。
