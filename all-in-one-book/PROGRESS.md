# 全书写作进度

## 状态说明
- ✅ 已完成并提交
- 🔄 进行中（当前轮次）
- ⏳ 待开始

## 进度表

| 章节 | 文件路径 | 状态 | 完成日期 |
|------|----------|------|----------|
| 第1章：系统全局架构 | 第一部分-架构全景/ch01-system-architecture.md | ✅ | 2026-03-27 |
| 第2章：入口与启动流程 | 第一部分-架构全景/ch02-entry-bootstrap.md | 🔄 | - |
| 第3章：类型系统与共享基础设施 | 第一部分-架构全景/ch03-types-infra.md | ⏳ | - |
| 第4章：Gateway 服务器 | 第二部分-Gateway控制平面/ch04-gateway.md | ⏳ | - |
| 第5章：消息路由引擎 | 第二部分-Gateway控制平面/ch05-routing.md | ⏳ | - |
| 第6章：Session 会话管理 | 第二部分-Gateway控制平面/ch06-sessions.md | ⏳ | - |
| 第7章：Pi Agent 核心 | 第三部分-Agent运行时/ch07-agent-runtime.md | ⏳ | - |
| 第8章：上下文引擎 | 第三部分-Agent运行时/ch08-context-engine.md | ⏳ | - |
| 第9章：工具系统 | 第三部分-Agent运行时/ch09-tools.md | ⏳ | - |
| 第10章：频道抽象层 | 第四部分-频道系统/ch10-channels.md | ⏳ | - |
| 第11章：典型频道实现 | 第四部分-频道系统/ch11-channel-impls.md | ⏳ | - |
| 第12章：媒体管道 | 第五部分-媒体与理解/ch12-media.md | ⏳ | - |
| 第13章：插件架构 | 第六部分-扩展与插件/ch13-plugins.md | ⏳ | - |
| 第14章：Skills 系统 | 第六部分-扩展与插件/ch14-skills.md | ⏳ | - |
| 第15章：安全模型 | 第七部分-安全系统/ch15-security.md | ⏳ | - |
| 第16章：macOS + Swabble | 第八部分-跨平台客户端/ch16-macos.md | ⏳ | - |
| 第17章：Android App | 第八部分-跨平台客户端/ch17-android.md | ⏳ | - |
| 第18章：Node 设备系统 | 第八部分-跨平台客户端/ch18-node-host.md | ⏳ | - |
| 第19章：Control UI 与 WebChat | 第九部分-前端/ch19-frontend.md | ⏳ | - |
| 第20章：Monorepo 与构建 | 第十部分-构建与工程/ch20-build.md | ⏳ | - |
| 第21章：关键路径追踪 | 第十一部分-端到端追踪/ch21-e2e-traces.md | ⏳ | - |
| 附录 | 附录/appendix.md | ⏳ | - |

## 下次续写指引

### 从哪里继续
- 下一章编号：第2章
- 下一章标题：入口与启动流程

### 已知待补充项
- ch01 `src/auto-reply/reply/commands-bash.ts` 的具体 tool handler 注册方式 [需源码验证]

### 质量基线回顾（下次续写前必读）
- 全书术语约定：Gateway=网关控制平面, Agent=智能体运行时, Channel=频道适配层, Session=会话, Binding=路由绑定, Plugin=插件, Hook=生命周期钩子, Lane=命令车道, Context Engine=上下文引擎
- 流程图风格：Mermaid sequenceDiagram，参与者用模块名（如 "WhatsApp Extension", "Gateway", "Router", "Agent"）
- 调用路径风格：`src/xxx.ts::functionName() → src/yyy.ts::functionName()`
- 语言：中文正文，技术术语保留英文（首次出现附中文解释）
