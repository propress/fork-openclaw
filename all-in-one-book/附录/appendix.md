# 附录

---

## 附录 A：src/ 模块速查表

| 目录 | 职责 | 章节 |
|------|------|------|
| `acp/` | Agent Communication Protocol | 第7章 |
| `agents/` | Agent 运行时核心 | 第7章 |
| `auto-reply/` | 自动回复管道 | 第1章, 第7章 |
| `bindings/` | 路由绑定 | 第5章 |
| `bootstrap/` | 启动引导 | 第2章 |
| `canvas-host/` | Canvas/A2UI 宿主 | 第9章 |
| `channels/` | 频道抽象层 | 第10章 |
| `chat/` | 聊天消息处理 | 第6章 |
| `cli/` | CLI 入口 + 路由 | 第2章 |
| `commands/` | 命令实现 | 第2章 |
| `compat/` | 兼容层 | — |
| `config/` | 配置系统 | 第3章 |
| `context-engine/` | 上下文引擎 | 第8章 |
| `cron/` | 定时任务 | 第1章, 第9章 |
| `daemon/` | 守护进程 | 第4章 |
| `extensions/` | 扩展加载 | 第13章 |
| `flows/` | 执行流编排 | 第7章 |
| `gateway/` | Gateway 控制平面 | 第4章 |
| `generated/` | 代码生成 | 第3章 |
| `hooks/` | Hook 生命周期 | 第13章 |
| `i18n/` | 国际化 | — |
| `image-generation/` | AI 图片生成 | 第9章 |
| `infra/` | 基础设施 | 第3章 |
| `interactive/` | 交互式 Session | 第6章 |
| `link-understanding/` | URL 理解 | 第12章 |
| `logging/` | 日志系统 | 第3章 |
| `markdown/` | Markdown 解析 | 第10章 |
| `media/` | 媒体存储/解析 | 第12章 |
| `media-understanding/` | 媒体理解 | 第12章 |
| `node-host/` | Node 设备宿主 | 第18章 |
| `pairing/` | DM 配对安全 | 第15章 |
| `plugin-sdk/` | 插件 SDK | 第13章 |
| `plugins/` | 插件运行时 | 第13章 |
| `process/` | 进程管理 + 命令队列 | 第4章 |
| `routing/` | 消息路由 | 第5章 |
| `secrets/` | 密钥管理 | 第15章 |
| `security/` | 安全模型 | 第15章 |
| `sessions/` | Session 管理 | 第6章 |
| `shared/` | 共享工具 | 第3章 |
| `terminal/` | 终端 UI | 第2章 |
| `tts/` | TTS 语音合成 | 第12章 |
| `tui/` | Terminal UI 框架 | 第2章 |
| `types/` | 类型定义 (.d.ts) | 第3章 |
| `utils/` | 通用工具 | 第3章 |
| `web-search/` | 网页搜索 | 第9章 |
| `wizard/` | 引导向导 | 第2章 |

---

## 附录 B：核心类型速查

| 类型 | 定义位置 | 说明 |
|------|---------|------|
| `MsgContext` | `src/auto-reply/types.ts` | 统一消息上下文 |
| `OpenClawConfig` | `src/config/types.openclaw.ts` | 主配置类型 |
| `AgentBinding` | `src/config/types.agents.ts` | 路由绑定 |
| `ResolvedAgentRoute` | `src/routing/resolve-route.ts` | 路由结果 |
| `SessionEntry` | `src/shared/session-types.ts` | Session 数据 |
| `ContextEngine` | `src/context-engine/types.ts` | 上下文引擎接口 |
| `PluginRegistry` | `src/plugins/types.ts` | 插件注册表 |
| `SecurityAuditReport` | `src/security/audit.ts` | 安全审计报告 |
| `QueueState` | `src/process/command-queue.ts` | 命令队列状态 |
| `GatewayServerOptions` | `src/gateway/server.impl.ts` | Gateway 配置 |

---

## 附录 C：WebSocket 协议列表

| 消息类型 | 方向 | 格式 | 说明 |
|---------|------|------|------|
| `call` | Client → Server | `{ type, id, method, params }` | RPC 调用 |
| `result` | Server → Client | `{ type, id, data }` | RPC 响应 |
| `push` | Server → Client | `{ type, event, data }` | 事件推送 |
| `error` | Server → Client | `{ type, id, error }` | 错误响应 |

常见 `method`：`chat:send`, `sessions:list`, `cron:list`, `config:get`, `agents:list`

常见 `event`：`agent:streaming`, `agent:complete`, `session:patch`, `cron:executed`, `channel:status`

---

## 附录 D：内置 Tool 注册表

| Tool | 风险级别 | 说明 |
|------|---------|------|
| `bash` | 🔴 高 | Shell 命令执行 |
| `browser` | 🔴 高 | 浏览器操作 |
| `message.send` | 🟡 中 | 主动发送消息 |
| `message.read` | 🟢 低 | 读取频道消息 |
| `cron.create` | 🟡 中 | 创建定时任务 |
| `cron.list` | 🟢 低 | 列出定时任务 |
| `web_search` | 🟢 低 | 网页搜索 |
| `image_generation` | 🟢 低 | AI 图片生成 |

---

## 附录 E：频道插件接口速查

| 接口 | 必须 | 说明 |
|------|------|------|
| `Inbound Monitor` | ✅ | 监听入站消息 |
| `Outbound Adapter` | ✅ | 发送出站消息 |
| `Setup Adapter` | 推荐 | 配置向导 |
| `Agent Tool` | 推荐 | message.send/read 能力 |
| `Health Monitor` | 可选 | 健康状态上报 |

---

## 附录 F：设计决策记录

| 决策 | 选择 | 替代方案 | 原因 |
|------|------|---------|------|
| 进程自重启 | spawn 子进程 | 原地修改 env | NODE_EXTRA_CA_CERTS 只读一次 |
| 命令队列存储 | `Symbol.for()` 全局 | 模块级变量 | 需跨 bundle chunk 共享 |
| 路由缓存 | WeakMap + 4000 上限 | LRU Cache | 单用户场景 4000 足够 |
| 快速路径 | 动态 import | 全量加载 | 启动速度优化 |
| Plugin SDK 边界 | 只允许 SDK 导入 | 允许 src/ 导入 | 稳定接口，核心可重构 |
| WebSocket 协议 | 自定义类 JSON-RPC | 标准 JSON-RPC 2.0 | 需要服务端主动推送 |
| Generation 机制 | 计数器 + 忽略旧结果 | 强制取消任务 | 避免中断导致数据不一致 |
| 媒体存储 | 临时文件 + TTL | 数据库存储 | 单用户，简单高效 |
| Lit 前端 | Web Components | React/Vue | bundle 更小，标准技术 |
| 本地语音识别 | Speech.framework | 云端 API | 隐私优先，零网络 |

---

## 附录 G：术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 网关 | Gateway | OpenClaw 的控制平面服务 |
| 智能体 | Agent | AI 对话执行体 |
| 频道 | Channel | 即时通讯平台适配 |
| 会话 | Session | 对话上下文存储单元 |
| 路由绑定 | Binding | 消息→Agent 的映射规则 |
| 车道 | Lane | 命令队列的并发控制单元 |
| 上下文引擎 | Context Engine | Prompt 组装和管理 |
| 压缩 | Compaction | 对话历史的智能摘要 |
| 插件 | Plugin | 可扩展的功能模块 |
| 钩子 | Hook | 生命周期事件处理器 |
| 媒体理解 | Media Understanding | 图片/音频/视频的 AI 分析 |
| 语音合成 | TTS (Text-to-Speech) | 文本转语音 |
| 唤醒词 | Wake Word | 语音激活触发词 |
| 沙箱 | Sandbox | 安全隔离执行环境 |
| 白名单 | Allowlist | 明确授权的访问控制 |
| 出站 | Outbound | 从 Agent 向外发送消息 |
| 入站 | Inbound | 从外部接收消息 |
| 信封 | Envelope | 消息的元数据包装 |
