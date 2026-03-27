# 第七章 · 插件系统

> **读完本章你将获得**：理解 OpenClaw 插件系统的完整生命周期 — 从发现、注册、加载到运行时 Hook 调用，以及 Plugin SDK 的公共 API 表面。

---

## 7.1 插件系统总览

OpenClaw 的插件系统是其可扩展性的核心。几乎所有非核心功能都以插件形式存在。

```mermaid
flowchart TB
    subgraph Discovery["发现 (Discovery)"]
        FS["扫描 extensions/* 目录"]
        MANIFEST["解析 openclaw.plugin.json"]
    end
    
    subgraph Registration["注册 (Registration)"]
        REGISTRY["Manifest Registry"]
        VALIDATE["兼容性验证 + 去重"]
    end
    
    subgraph Loading["加载 (Loading)"]
        INSTALL["npm install --omit=dev"]
        IMPORT["动态 import() 入口"]
        RUNTIME["创建 PluginRuntime"]
    end
    
    subgraph Execution["运行 (Execution)"]
        HOOKS["Hook 注册"]
        ADAPTERS["适配器注册"]
        TOOLS["工具注册"]
    end
    
    Discovery --> Registration --> Loading --> Execution
```

---

## 7.2 插件声明文件

每个插件通过 `openclaw.plugin.json` 声明自己的身份和能力：

```json
{
  "id": "openai",
  "version": "1.0.0",
  "openclaw": {
    "install": {
      "npmSpec": "@openclaw/openai"
    },
    "providers": ["openai"],
    "auth": {
      "choices": [
        { "type": "api-key", "envVar": "OPENAI_API_KEY" }
      ]
    },
    "cli": {
      "backends": ["openai"]
    }
  }
}
```

### 声明字段

| 字段 | 用途 |
|------|------|
| `id` | 插件唯一标识 |
| `openclaw.install.npmSpec` | npm 安装包名 |
| `openclaw.providers` | 提供的 LLM Provider 名称 |
| `openclaw.channel.id` | 提供的 Channel ID |
| `openclaw.auth` | 认证方式声明 |
| `openclaw.cli.backends` | CLI 后端名称 |

---

## 7.3 插件发现

```mermaid
flowchart TB
    A["discoverPlugins(workspaceDir, loadPaths)"] --> B["扫描目录"]
    B --> C["extensions/openai/"]
    B --> D["extensions/telegram/"]
    B --> E["extensions/.../"]
    
    C --> F["读取 openclaw.plugin.json"]
    D --> F
    E --> F
    
    F --> G["解析 Manifest"]
    G --> H["版本兼容性检查"]
    H --> I["返回 Plugin Manifest 数组"]
```

**关键代码**：`src/plugins/discovery.ts::discoverPlugins()`

扫描路径包括：
1. 内置 `extensions/` 目录
2. 配置中指定的额外加载路径 (`config.plugins.load.paths`)

---

## 7.4 Manifest Registry

```typescript
// src/plugins/manifest-registry.ts
function buildPluginManifestRegistry(params: {
  workspace: string;
  loadPaths: string[];
  config: OpenClawConfig;
}): PluginManifestRegistry

type PluginManifestRegistry = {
  get(id: string): PluginManifest | undefined;
  list(): PluginManifest[];
  getEnabled(): PluginManifest[];
  isEnabled(id: string): boolean;
};
```

Registry 维护所有发现的插件清单，并根据配置决定哪些启用。

---

## 7.5 插件加载与运行时

### 加载流程

```mermaid
sequenceDiagram
    participant GW as Gateway
    participant BOOT as Plugin Bootstrap
    participant FS as 文件系统
    participant RT as Plugin Runtime

    GW->>BOOT: loadGatewayStartupPlugins()
    
    loop 对每个已启用的插件
        BOOT->>FS: npm install --omit=dev<br/>(在插件目录下)
        BOOT->>FS: 动态 import() 插件入口
        FS-->>BOOT: 插件模块 (default export)
        BOOT->>RT: createPluginRuntime()
        
        alt ChannelPlugin
            RT->>GW: 注册 Channel 适配器到 Channel Manager
        else ProviderPlugin
            RT->>GW: 注册 Provider 到 Model Catalog
        else ToolPlugin
            RT->>GW: 注册工具到 Tool Registry
        end
    end
```

**关键代码**：
```
src/gateway/server-plugin-bootstrap.ts::loadGatewayStartupPlugins()
src/plugins/runtime/index.ts::createPluginRuntime()
```

### PluginRuntime API

```typescript
// src/plugins/runtime/index.ts
type PluginRuntime = {
  agent: AgentRuntimeApi;          // Agent 运行时操作
  channel: ChannelRuntimeApi;      // 通道通信
  media: MediaRuntimeApi;          // 媒体理解
  subagent: SubagentRuntimeApi;    // 子 Agent
  config: ConfigRuntimeApi;        // 配置访问
  system: SystemRuntimeApi;        // 系统调用
};
```

---

## 7.6 Hook 系统

插件通过 Hook 在关键时刻注入行为，无需修改核心代码。

```mermaid
flowchart LR
    subgraph Hooks["可用 Hook"]
        H1["beforeAgentStart<br/>Agent 开始前"]
        H2["llmInputHook<br/>修改 LLM 输入"]
        H3["llmOutputHook<br/>观察 LLM 输出"]
        H4["sessionMessageHook<br/>修改会话消息"]
        H5["gateway.onShutdown<br/>Gateway 关闭时"]
        H6["gateway_stop<br/>Gateway 停止"]
    end
```

### Hook 调用时机

| Hook | 调用时机 | 能力 |
|------|---------|------|
| `beforeAgentStart` | Agent 执行前 | 修改配置、注入上下文 |
| `llmInputHook` | LLM 请求发出前 | **修改** prompt、messages、tools |
| `llmOutputHook` | LLM 响应收到后 | **观察** 输出（只读） |
| `sessionMessageHook` | 消息保存到 Session 前 | **修改** 消息内容 |
| `gateway.onShutdown` | Gateway 关闭时 | 清理资源 |

**关键代码**：`src/plugins/hooks.ts`

---

## 7.7 Plugin SDK 公共表面

Plugin SDK 是插件开发者的唯一合法依赖。它定义了所有插件可用的类型和接口。

```mermaid
flowchart TB
    subgraph SDK["src/plugin-sdk/ (199 文件)"]
        CORE["core.ts<br/>Channel 核心类型"]
        INDEX["index.ts<br/>主入口导出"]
        CHAN["channel-*.ts<br/>通道特定 Helper"]
        PROV["provider-entry.ts<br/>Provider 接口"]
        TOOL["tool-send.ts<br/>工具调用 Helper"]
        WEBHOOK["webhook-*.ts<br/>Webhook 入口"]
        MEDIA["outbound-media.ts<br/>出站媒体"]
        IMG["image-generation.ts<br/>图像生成"]
        SECRET["secret-input.ts<br/>凭证输入 Schema"]
    end
```

### 跨包引用规则

| 引用方向 | 允许 | 路径 |
|---------|------|------|
| Extension → Plugin SDK | ✅ | `openclaw/plugin-sdk/*` |
| Extension → 本地 barrel | ✅ | `./api.ts`, `./runtime-api.ts` |
| Extension → Core `src/**` | ❌ | 禁止 |
| Extension → 另一个 Extension | ❌ | 禁止 |

---

## 7.8 插件类型一览

| 类型 | 数量 | 示例 |
|------|------|------|
| **LLM Provider** | 35+ | openai, anthropic, google, ollama, groq, mistral |
| **Channel** | 20+ | telegram, discord, slack, matrix, irc, msteams |
| **媒体理解** | 3+ | deepgram (STT), elevenlabs (TTS), microsoft (Speech) |
| **记忆** | 2 | memory-core, memory-lancedb |
| **工具** | 10+ | browser, duckduckgo, tavily, brave, firecrawl |
| **其他** | 10+ | lobster, diagnostics-otel, copilot-proxy, voice-call |

---

### 质检报告

**完整性**
- [x] 发现 → 注册 → 加载 → 运行时全生命周期
- [x] openclaw.plugin.json 声明文件
- [x] Manifest Registry
- [x] PluginRuntime API
- [x] Hook 系统与调用时机
- [x] Plugin SDK 公共表面与引用规则
- [x] 插件类型一览

**准确性**
- [x] 89+ 插件数量已确认
- [x] 引用规则与 AGENTS.md 一致
- [x] Hook 名称与 hooks.ts 一致

**可读性**
- [x] 从总览 → 声明 → 发现 → 加载 → Hook → SDK，递进清晰
- [x] 每个阶段有图

**勘误建议**
- 无
