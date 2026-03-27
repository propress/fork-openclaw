# OpenClaw 实现原理全解 — 写作进度

## 章节规划

| # | 章节标题 | 文件名 | 核心覆盖 | 状态 |
|---|---------|--------|---------|------|
| 1 | 序言：全局视角 | ch01-overview.md | 项目定位、设计哲学、架构全景图、核心概念词典、代码库地图、典型交互极简全流程 | ⏳ |
| 2 | 数据流全景 | ch02-data-flows.md | 消息接收、Agent 处理、消息发送、媒体流转、插件调用、Gateway 生命周期等典型场景的完整数据流 | ⏳ |
| 3 | Gateway：系统心脏 | ch03-gateway.md | HTTP/WS 服务器、RPC 协议、方法注册、连接认证、配置热重载、发现服务、健康监控 | ⏳ |
| 4 | Agent 引擎：从提示到回答 | ch04-agent-engine.md | PI Embedded Runner、执行循环、重试/回退、System Prompt 构建、流式输出、使用量追踪 | ⏳ |
| 5 | Channel 系统：多平台消息适配 | ch05-channels.md | Channel Plugin 架构、适配器体系、消息归一化、能力声明、线程绑定、出站适配 | ⏳ |
| 6 | 路由与会话 | ch06-routing-sessions.md | 路由决策树、Binding 匹配、Session Key 构建、会话生命周期、转录持久化 | ⏳ |
| 7 | 插件系统 | ch07-plugin-system.md | 发现→注册→加载→执行、Plugin SDK 公共表面、Hook 机制、运行时 API、插件配置 Schema | ⏳ |
| 8 | 配置体系 | ch08-config.md | Zod Schema 体系、配置加载/合并、热重载计划、配置文档生成、漂移检测 | ⏳ |
| 9 | 媒体管道 | ch09-media.md | 图片/音频/视频/PDF 处理、FFmpeg 集成、媒体存储与服务、入站/出站媒体路径 | ⏳ |
| 10 | 安全体系 | ch10-security.md | 安全审计框架、执行审批、路径守卫、沙箱隔离、通道安全策略、密钥管理 | ⏳ |
| 11 | 命令系统 (CLI) | ch11-cli.md | Commander 骨架、命令注册、Doctor 诊断、Onboard 流程、交互式向导 | ⏳ |
| 12 | Web UI 控制面 | ch12-web-ui.md | Lit.js 架构、Gateway WebSocket 连接、聊天流式渲染、Canvas Host | ⏳ |
| 13 | 原生客户端 | ch13-native-apps.md | iOS (Swift/SwiftUI)、Android (Kotlin/Compose)、macOS 应用、设备配对协议 | ⏳ |
| 14 | 上下文引擎与压缩 | ch14-context-engine.md | Context Engine 接口、注册/解析、Token 预算、压缩策略、Legacy 引擎、自定义引擎 | ⏳ |
| 15 | 模型与 Provider | ch15-models-providers.md | Model Catalog、Auth Profile 轮转、Provider 插件、模型回退链、动态模型解析、定价缓存 | ⏳ |
| 16 | 工具与技能 | ch16-tools-skills.md | Bash 工具、Agent 工具注册、技能快照、工具 Schema 约束、执行审批流 | ⏳ |
| 17 | 子 Agent 与编排 | ch17-subagents.md | Subagent Registry、Spawn 生命周期、父子通信、完成公告、层级会话 | ⏳ |
| 18 | 基础设施层 | ch18-infra.md | 设备认证/配对、心跳、进程管理、二进制发现、文件安全、更新机制、Bonjour 发现 | ⏳ |
| 19 | 构建与测试 | ch19-build-test.md | tsdown 构建、Vitest 测试体系、覆盖率门槛、E2E 测试、pnpm Workspace、CI 流水线 | ⏳ |
| 20 | 端到端追踪 | ch20-e2e-traces.md | 场景 1: Telegram 消息→Agent 回复全路径; 场景 2: Web UI 聊天流式回复; 场景 3: 插件加载→工具调用 | ⏳ |

## 状态说明
- ✅ 已完成  - 🔄 进行中  - ⏳ 待开始

## 术语约定

| 英文术语 | 中文说明 | 首次出现 |
|---------|---------|---------|
| Gateway | 网关 — 系统核心进程，承载 HTTP/WS 服务、管理 Channel 和 Agent | ch01 |
| Agent | 代理 — LLM 驱动的对话实体，处理消息并生成回复 | ch01 |
| Channel | 通道 — 消息平台适配器（Telegram、Discord 等） | ch01 |
| Plugin | 插件 — 可扩展模块，提供 Channel、Provider 或工具能力 | ch01 |
| Session | 会话 — Agent 与用户的持久对话上下文 | ch01 |
| Session Key | 会话键 — 唯一标识一个会话的层级字符串 | ch01 |
| Pi / PI | 嵌入式 Agent 运行时（Project Intelligence） | ch01 |
| Binding | 绑定 — Channel/Account/Peer 到 Agent 的路由映射规则 | ch01 |
| Provider | 提供者 — LLM API 接入层（OpenAI、Anthropic 等） | ch01 |
| Auth Profile | 认证配置 — 包含 API Key 和访问策略的凭证组 | ch01 |
| Context Engine | 上下文引擎 — 管理 Agent 对话历史的 Token 预算和压缩 | ch01 |
| Subagent | 子代理 — 由父 Agent 生成的子任务执行者 | ch01 |
| Node | 节点 — 已配对的远程设备（iOS、Android 等） | ch01 |
| Skill | 技能 — Agent 可调用的工具/脚本包 | ch01 |
| Hook | 钩子 — 插件可注入的生命周期回调 | ch01 |
| RPC | 远程过程调用 — Gateway WebSocket 上的请求-响应协议 | ch01 |

## 下次续写指引

### 从哪里继续
从 Chapter 1 (ch01-overview.md) 开始写。

### 交接备忘
- 项目探索已完成，对架构、数据流、模块职责有全局理解
- 需注意：extensions/ 目录包含 89+ 插件，按 Provider / Channel / Utility 三大类分
- Gateway 暴露 114+ RPC 方法，Protocol Version 3
- Agent 执行核心在 `src/agents/pi-embedded-runner/run.ts`（1400+ 行）
- 路由决策为多层优先级匹配（Peer+Roles → Guild → Account+Peer → Channel → Default）

### 待验证项
- [ ] Gateway 协议 Frame 类型的完整枚举
- [ ] Auth Profile 轮转的精确触发条件
- [ ] Context Engine Legacy 策略的 Token 阈值默认值
