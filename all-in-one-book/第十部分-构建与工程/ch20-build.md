# 第20章：Monorepo 与构建

> **一句话收获**：读完本章，你将理解 OpenClaw 的"工程骨架"——pnpm workspace 如何组织 80+ 包、tsdown 如何构建 TypeScript、以及 12 步构建管道的每一步做了什么。

---

## 20.1 Monorepo 结构

### 20.1.1 一句话理解

**OpenClaw 的 Monorepo 就像一个"工业园区"**——主厂房（`src/`）负责核心产品，80+ 个卫星工厂（`extensions/`）生产各种配件，几个专业车间（`packages/`）做特殊部件，前端展厅（`ui/`）展示产品。

### 20.1.2 Workspace 布局

```mermaid
flowchart TD
    subgraph Root["pnpm Workspace 根"]
        SRC["src/<br/>核心源码"]
        DIST["dist/<br/>构建输出"]
    end

    subgraph Extensions["extensions/ (80+)"]
        EXT_DISCORD["discord"]
        EXT_WHATSAPP["whatsapp"]
        EXT_ANTHROPIC["anthropic"]
        EXT_MORE["... 80+ 扩展"]
    end

    subgraph Packages["packages/"]
        CLAWDBOT["clawdbot<br/>(CLI 兼容 shim)"]
        MEMORY["memory-host-sdk<br/>(内存/嵌入 SDK)"]
        MOLTBOT["moltbot<br/>(CLI 兼容 shim)"]
    end

    subgraph UI_WS["ui/"]
        LIT_APP["Lit Web Components"]
    end

    subgraph Apps["apps/"]
        ANDROID["android/"]
        MACOS["macos/"]
        IOS["ios/"]
    end
```

---

## 20.2 构建管道

### 20.2.1 12 步构建

`pnpm build` 执行以下 12 个步骤：

```mermaid
flowchart TD
    B1["1. canvas:a2ui:bundle<br/>打包 Canvas UI"]
    B2["2. tsdown-build.mjs<br/>TypeScript 编译"]
    B3["3. runtime-postbuild.mjs<br/>运行时模块后处理"]
    B4["4. build-stamp.mjs<br/>添加构建元数据"]
    B5["5. build:plugin-sdk:dts<br/>生成 Plugin SDK .d.ts"]
    B6["6. write-plugin-sdk-entry-dts.ts<br/>写入 SDK 类型入口"]
    B7["7. canvas-a2ui-copy.ts<br/>复制 Canvas 资源"]
    B8["8. copy-hook-metadata.ts<br/>复制 Hook 元数据"]
    B9["9. copy-export-html-templates.ts<br/>导出 HTML 模板"]
    B10["10. write-build-info.ts<br/>写入构建信息"]
    B11["11. write-cli-startup-metadata.ts<br/>CLI 启动元数据"]
    B12["12. write-cli-compat.ts<br/>CLI 兼容层"]

    B1 --> B2 --> B3 --> B4 --> B5 --> B6 --> B7 --> B8 --> B9 --> B10 --> B11 --> B12
```

### 20.2.2 tsdown 配置

```
tsdown.config.ts:
├── 入口: src/index.ts, src/entry.ts, CLI, runtime 模块
├── 平台: Node.js
├── 格式: ESM
├── Plugin SDK 子路径单独打包
├── 内置 Hook 从 src/hooks/bundled/ 加载
└── 压制 eval 警告 (protobufjs, bottleneck)
```

---

## 20.3 工具链

| 工具 | 用途 | 命令 |
|------|------|------|
| **pnpm** | 包管理 + Workspace | `pnpm install` |
| **tsdown** | TypeScript 编译 | `pnpm build` |
| **Oxlint** | Lint | `pnpm lint` |
| **Oxfmt** | 格式化 | `pnpm format` |
| **tsgo** | 类型检查 | `pnpm tsgo` |
| **Vitest** | 测试 | `pnpm test` |
| **Vite** | UI 构建 | `ui/vite.config.ts` |

---

## 20.4 开发命令速查

| 命令 | 说明 |
|------|------|
| `pnpm install` | 安装依赖 |
| `pnpm build` | 完整构建 |
| `pnpm dev` | 开发模式 |
| `pnpm check` | Lint + Format + Types |
| `pnpm test` | 运行测试 |
| `pnpm test:coverage` | 测试 + 覆盖率 |
| `pnpm gateway:dev` | Gateway 开发模式 |
| `pnpm openclaw ...` | CLI 开发模式执行 |

---

## 质检报告

### 自检 1：完整性
- [x] 覆盖 Monorepo 结构、构建管道、工具链
- [x] 流程图数量：2 张

### 自检 2：准确性
- [x] 12 步构建基于 package.json scripts
- [x] 工具链基于实际配置文件

### 自检 3：可读性
- [x] "工业园区"类比直观
- [x] 命令速查表实用
