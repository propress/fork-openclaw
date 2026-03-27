# 第21章：关键路径追踪

> **一句话收获**：读完本章，你将能在脑中完整"回放"五条端到端链路——从用户发消息到 Agent 回复的每一个函数调用、每一次模块跳转。

---

## 21.1 WhatsApp "Hello" 全链路

### 完整调用链（26 步）

```mermaid
sequenceDiagram
    participant User as 用户 WhatsApp
    participant Baileys as Baileys SDK
    participant OnMsg as on-message.ts
    participant Process as process-message.ts
    participant Route as resolve-route.ts
    participant Queue as command-queue.ts
    participant AgentCmd as agent-command.ts
    participant RunCtx as run-context.ts
    participant Attempt as attempt-execution.ts
    participant CtxEngine as Context Engine
    participant LLM as LLM Provider
    participant Delivery as delivery.ts
    participant Outbound as outbound/deliver.ts
    participant SendAPI as send-api.ts

    User->>Baileys: WhatsApp 消息 "Hello"
    Baileys->>OnMsg: createWebOnMessageHandler()
    OnMsg->>OnMsg: resolvePeerId()
    OnMsg->>OnMsg: Echo 检测 (过滤自己的消息)
    OnMsg->>Process: processMessage()
    Process->>Process: 构建 MsgContext { Body, From, Channel, ChatType }
    Process->>Route: resolveAgentRoute({ channel: "whatsapp", peer })
    Route->>Route: 七级优先级匹配
    Route-->>Process: { agentId: "main", sessionKey, matchedBy }
    Process->>Queue: enqueueCommandInLane("main", agentCommand)
    Queue->>AgentCmd: executeAgentCommand({ body: "Hello", sessionKey })
    AgentCmd->>AgentCmd: loadSessionStore(sessionKey)
    AgentCmd->>RunCtx: resolveAgentConfig()
    RunCtx-->>AgentCmd: { model, provider, skills }
    AgentCmd->>Attempt: runAgentAttempt()
    Attempt->>CtxEngine: assemble({ messages, tokenBudget })
    CtxEngine-->>Attempt: { messages: [...], estimatedTokens }
    Attempt->>LLM: Chat Completion 请求
    LLM-->>Attempt: "Hi! How can I help you?"
    Attempt-->>AgentCmd: AgentRunResult { payloads }
    AgentCmd->>AgentCmd: persistSessionEntry()
    AgentCmd->>Delivery: deliverAgentCommandResult()
    Delivery->>Outbound: deliverOutboundPayloads()
    Outbound->>SendAPI: sendMessage(jid, text)
    SendAPI->>Baileys: sock.sendMessage()
    Baileys->>User: "Hi! How can I help you?"
```

---

## 21.2 Cron → isolated Agent → announce

```mermaid
sequenceDiagram
    participant Timer as Cron Timer
    participant CronSvc as CronService
    participant Queue as command-queue.ts
    participant IsoAgent as 隔离 Agent
    participant Session as 临时 Session
    participant LLM as LLM
    participant Dispatch as delivery-dispatch.ts
    participant Outbound as outbound/deliver.ts
    participant Channel as 频道适配器

    Timer->>CronSvc: 时间匹配 "0 9 * * *"
    CronSvc->>Queue: enqueueCommandInLane("cron", cronJob)
    Queue->>IsoAgent: runCronAgentTurn({ payload })
    IsoAgent->>Session: 创建隔离 Session
    IsoAgent->>LLM: Agent Loop (可能含工具调用)
    LLM-->>IsoAgent: 执行结果文本
    IsoAgent-->>Dispatch: RunCronAgentTurnResult
    Dispatch->>Dispatch: resolveDeliveryTarget()
    Dispatch->>Outbound: deliverOutboundPayloads()
    Outbound->>Channel: 发送到 announce 频道
    Channel-->>Outbound: 发送成功
```

---

## 21.3 Browser Tool

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as Agent
    participant LLM as LLM
    participant BrowserTool as Browser Tool
    participant Sandbox as 浏览器沙箱

    User->>Agent: "帮我打开 example.com"
    Agent->>LLM: [上下文 + 用户消息]
    LLM-->>Agent: tool_use: browser({ url: "example.com" })
    Agent->>BrowserTool: 执行浏览器操作
    BrowserTool->>Sandbox: 启动沙箱浏览器 (headless)
    Sandbox->>Sandbox: 导航到 URL
    Sandbox->>Sandbox: 截取页面内容/截图
    Sandbox-->>BrowserTool: 页面内容 + 截图
    BrowserTool-->>Agent: tool_result: { content, screenshot }
    Agent->>LLM: [上下文 + 工具结果]
    LLM-->>Agent: "这个网站的内容是..."
    Agent-->>User: 回复
```

---

## 21.4 图片 → Vision → 回复

```mermaid
sequenceDiagram
    participant User as 用户
    participant Channel as 频道扩展
    participant MU as media-understanding
    participant Vision as Vision API
    participant Agent as Agent
    participant LLM as LLM

    User->>Channel: 图片 + "这是什么？"
    Channel->>Channel: 提取 MediaPath, 构建 MsgContext
    Channel->>MU: applyMediaUnderstanding()
    MU->>MU: normalizeMediaAttachments()
    MU->>MU: resolveAttachmentKind() → "image"
    MU->>Vision: Vision API 调用 (图片 → 文本描述)
    Vision-->>MU: "一只橘猫在沙发上睡觉"
    MU-->>Channel: 注入 <image>描述</image> 到上下文
    Channel->>Agent: executeAgentCommand({ body, mediaContext })
    Agent->>LLM: [上下文 + 图片描述 + 用户问题]
    LLM-->>Agent: "这是一只橘猫，看起来睡得很香！"
    Agent-->>User: 回复
```

---

## 21.5 Webhook → Agent turn → 投递

```mermaid
sequenceDiagram
    participant Ext as 外部服务
    participant HTTP as Gateway HTTP
    participant Auth as 认证检查
    participant Dispatch as Hook 调度
    participant Agent as 隔离 Agent
    participant LLM as LLM
    participant Outbound as 出站投递
    participant Channel as 频道

    Ext->>HTTP: POST /hooks/agent { message, agentId, deliver }
    HTTP->>Auth: extractHookToken() + safeEqualSecret()
    Auth-->>HTTP: ✅ 认证通过
    HTTP->>HTTP: normalizeAgentPayload()
    HTTP->>HTTP: 幂等性检查 (replay cache)
    HTTP->>Dispatch: dispatchAgentHook()
    Dispatch->>Agent: 创建隔离 Agent turn
    Agent->>LLM: Agent Loop 执行
    LLM-->>Agent: 回复文本
    Agent->>Outbound: deliverOutboundPayloads()
    Outbound->>Channel: 发送到指定频道
    Channel-->>Outbound: 发送成功
    HTTP-->>Ext: 200 OK { status: "dispatched" }
```

---

## 质检报告

### 自检 1：完整性
- [x] 覆盖全部 5 条端到端路径
- [x] 流程图数量：5 张

### 自检 2：准确性
- [x] WhatsApp 全链路基于源码逐步追踪
- [x] 各路径与前序章节的调用链一致

### 自检 3：可读性
- [x] 每条路径独立可读
- [x] 序列图展示完整的参与者和调用顺序
