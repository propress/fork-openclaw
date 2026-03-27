# 第十五章 · 模型与 Provider

> **读完本章你将获得**：理解 OpenClaw 如何接入 35+ LLM 提供者 — Model Catalog、Provider 插件接口、Auth Profile 轮转、模型回退链和动态模型解析。

---

## 15.1 Provider 体系总览

```mermaid
flowchart TB
    subgraph Providers["35+ Provider 插件"]
        OAI["openai"]
        ANTH["anthropic"]
        GOOG["google"]
        OLL["ollama"]
        GROQ["groq"]
        MISTRAL["mistral"]
        DS["deepseek"]
        MORE["...30 more"]
    end
    
    subgraph Core["核心层"]
        CATALOG["Model Catalog<br/>src/agents/model-catalog.ts"]
        AUTH["Auth Profiles<br/>src/agents/auth-profiles/"]
        SELECT["Model Selection<br/>src/agents/model-selection.ts"]
        FALLBACK["Model Fallback<br/>src/agents/model-fallback.ts"]
        PRICING["Pricing Cache<br/>src/agents/model-pricing-cache.ts"]
    end
    
    Providers -->|"providerCatalog Hook"| CATALOG
    CATALOG --> SELECT
    AUTH --> SELECT
    SELECT --> FALLBACK
```

---

## 15.2 Provider 插件接口

```typescript
// src/plugin-sdk/provider-entry.ts (简化)
type OpenClawPluginApi = {
  // 模型目录贡献
  providerCatalog?: (ctx) => ProviderCatalogResult;
  
  // 动态模型准备
  providerPrepareDynamicModel?: (ctx) => Promise<PreparedModel>;
  
  // 模型解析
  providerResolveModel?: (ctx) => Promise<ResolvedModel>;
  
  // 运行时认证准备
  providerPrepareRuntimeAuth?: (ctx) => Promise<PreparedAuth>;
  
  // 流式包装
  providerWrapStream?: (ctx) => AsyncIterable<StreamChunk>;
};
```

每个 Provider 插件实现这些 Hook 来告诉核心"我有哪些模型"和"怎么调用它们"。

---

## 15.3 Model Catalog

Model Catalog 是所有可用模型的注册表：

```mermaid
flowchart LR
    subgraph Sources["数据来源"]
        BUILTIN["内置模型列表"]
        PLUGINS["Provider 插件贡献<br/>(providerCatalog Hook)"]
    end
    
    MERGE["合并 + 去重"] --> CATALOG["Model Catalog"]
    Sources --> MERGE
    
    CATALOG --> QUERY["查询: provider + modelName"]
    CATALOG --> LIST["列表: 所有可用模型"]
    CATALOG --> ALIAS["别名: fast → gpt-4-mini"]
```

### 模型别名

```yaml
# config.yaml
models:
  aliases:
    fast: gpt-4-mini
    smart: claude-sonnet
    local: ollama/llama3
```

用户可以用简短别名指代具体模型。

---

## 15.4 Auth Profile 管理

```mermaid
flowchart TB
    subgraph Profiles["Auth Profiles"]
        P1["Profile A<br/>OpenAI Key #1"]
        P2["Profile B<br/>OpenAI Key #2"]
        P3["Profile C<br/>OpenAI Key #3"]
    end
    
    SELECT["选择 Profile"] --> P1
    P1 -->|"成功"| USE["使用"]
    P1 -->|"401/429"| COOLDOWN1["冷却"]
    COOLDOWN1 --> P2
    P2 -->|"成功"| USE
    P2 -->|"失败"| P3
```

### Profile 生命周期

| 状态 | 含义 |
|------|------|
| Active | 可用，首选 |
| Cooldown | 认证失败/限流，等待恢复 |
| Exhausted | 所有 Profile 都不可用 |

**关键代码**：
```
src/agents/auth-profiles/
  ├── rotation.ts    — 轮转逻辑
  ├── cooldown.ts    — 冷却期管理
  ├── storage.ts     — 持久化
  └── doctor.ts      — 诊断
```

---

## 15.5 模型回退链

当首选模型不可用时，按配置的 fallback 链降级：

```yaml
agents:
  - id: default
    model: claude-sonnet
    models:
      fallback:
        - gpt-4              # 第一备选
        - ollama/llama3       # 第二备选（本地）
```

```mermaid
flowchart LR
    A["claude-sonnet<br/>(首选)"] -->|"不可用"| B["gpt-4<br/>(备选 1)"]
    B -->|"不可用"| C["ollama/llama3<br/>(备选 2)"]
    C -->|"不可用"| D["❌ 报错"]
```

---

## 15.6 定价缓存

```
src/agents/model-pricing-cache.ts
```

维护各模型的输入/输出 Token 单价，用于 `UsageAccumulator` 中的费用估算。

---

### 质检报告

**完整性**
- [x] Provider 插件接口
- [x] Model Catalog 与别名
- [x] Auth Profile 轮转
- [x] 模型回退链
- [x] 定价缓存

**准确性**
- [x] 接口与 plugin-sdk 一致

**可读性**
- [x] 递进清晰，图表辅助理解

**勘误建议**
- 无
