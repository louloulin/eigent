# Claude Code、OpenCode 等工具集成协议分析

## 📋 文档概述

**文档版本**: 1.0
**创建日期**: 2025-02-06
**主题**: 多智能体协议对比分析 - MCP vs A2A vs ACP
**目标**: 为 Eigent 项目选择最佳的多智能体集成协议

---

## 📊 目录

1. [背景分析](#1-背景分析)
2. [三大协议深度对比](#2-三大协议深度对比)
3. [Claude Code & OpenCode 现状](#3-claude-code--opencode-现状)
4. [协议选择建议](#4-协议选择建议)
5. [混合架构方案](#5-混合架构方案)
6. [实施路线图](#6-实施路线图)
7. [参考资料](#参考资料)

---

## 1. 背景分析

### 1.1 多智能体协议的兴起

2024-2025 年，AI 智能体领域出现了多个标准化协议，旨在解决智能体之间的互操作性问题：

| 协议 | 发布时间 | 提出方 | 核心目标 |
|------|---------|--------|---------|
| **MCP** | 2024年5月 | Anthropic | 模型上下文扩展 |
| **ACP** | 2025年3月 | IBM BeeAI | 智能体间通信 |
| **A2A** | 2025年4月 | Google | 智能体对等协作 |

### 1.2 2025年8月重大变化

**ACP 与 A2A 合并**

- 2025年8月29日，IBM ACP 团队与 Google A2A 团队宣布合并
- 在 Linux Foundation 的 LF AI & Data 基金会下开发统一标准
- ACP 团队停止积极开发，将技术和专业知识贡献给 A2A

**影响**：
- ✅ 避免协议分裂，形成统一标准
- ✅ 整合 IBM 和 Google 的生态系统
- ⚠️ ACP 项目进入维护模式

### 1.3 Eigent 项目的集成需求

基于架构分析，Eigent 需要集成：

1. **Claude Code** - Anthropic 的 AI 编程助手
2. **OpenCode** - 开源 AI 编程工具
3. **CodeX** - 其他 AI 编程工具
4. **未来更多智能体** - Qwen Code、通义灵码等

---

## 2. 三大协议深度对比

### 2.1 核心特性对比表

| 特性维度 | **MCP** | **ACP** | **A2A** |
|---------|---------|---------|---------|
| **全称** | Model Context Protocol | Agent Communication Protocol | Agent-to-Agent Protocol |
| **提出方** | Anthropic | IBM BeeAI | Google |
| **治理模式** | Anthropic 主导 | 开放治理（LF AI & Data） | Google + 50+ 合作伙伴 |
| **核心焦点** | 单模型 + 多工具 | 多智能体通信 | 多智能体对等协作 |
| **通信方式** | JSON-RPC 2.0 | RESTful API | (待公开) |
| **SDK 要求** | 必需 | 可选 | (待定) |
| **流式支持** | 基础流式 | Delta 流式（细粒度） | 高级流式 |
| **消息结构** | 任意 JSON Schema | 标准化消息格式 | 结构化消息 |
| **状态管理** | 无状态 | 支持有状态 | 全面状态管理 |
| **发现机制** | 客户端配置 | 在线/离线发现 | 动态发现 |
| **安全性** | 基础 | 企业级 | 企业级+ |
| **生态系统** | Anthropic Claude | IBM BeeAI | Google AI Studio |
| **成熟度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **文档质量** | 优秀 | 良好 | 优秀 |

### 2.2 详细技术对比

#### MCP (Model Context Protocol)

**优势**：
- ✅ 成熟稳定，已有大量实现
- ✅ Anthropic Claude 原生支持
- ✅ 丰富的服务器生态系统
- ✅ 简单的 JSON-RPC 实现
- ✅ 专注工具调用，场景明确

**劣势**：
- ❌ 不支持智能体间直接通信
- ❌ 缺乏状态管理
- ❌ 流式支持有限
- ❌ 消息格式过于灵活，互操作性差
- ❌ 不适合多智能体协作

**适用场景**：
- 单个 LLM 访问外部工具和数据
- 工具集成（文件系统、API、数据库）
- 简单的上下文扩展

#### ACP (Agent Communication Protocol)

**优势**：
- ✅ RESTful API，无需专用 SDK
- ✅ 支持所有模态（MimeTypes）
- ✅ 在线/离线智能体发现
- ✅ 异步优先，同步支持
- ✅ 标准化消息格式
- ✅ 开放治理

**劣势**：
- ❌ 项目已合并到 A2A，进入维护模式
- ❌ 生态系统相对较小
- ❌ 实现案例较少
- ❌ 长期维护风险

**适用场景**：
- 多智能体协作（但建议转向 A2A）
- 跨框架集成
- 需要简单 REST 接口的场景

#### A2A (Agent-to-Agent Protocol)

**优势**：
- ✅ Google 强力支持，50+ 合作伙伴
- ✅ 全面的状态管理
- ✅ 高级流式支持
- ✅ 企业级安全性
- ✅ 与 Google 生态深度集成
- ✅ 未来统一标准的潜力
- ✅ 支持动态协作和自主决策

**劣势**：
- ❌ 规范尚未完全公开
- ❌ 相对较新，实现较少
- ❌ 可能偏向 Google 生态
- ❌ 学习曲线可能较陡

**适用场景**：
- 复杂的多智能体协作
- 需要状态管理的长期任务
- 跨组织智能体协作
- 企业级部署

### 2.3 协议层次关系

```
┌─────────────────────────────────────────────────┐
│           应用层 (Application Layer)             │
│  Claude Code | OpenCode | CodeX | Custom Agents  │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│          智能体通信层 (Agent Communication)       │
│                                                 │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐ │
│    │   A2A    │    │   ACP    │    │Custom    │ │
│    │ (推荐)   │    │ (维护中) │    │Protocol  │ │
│    └──────────┘    └──────────┘    └──────────┘ │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│           工具访问层 (Tool Access)               │
│                                                 │
│    ┌──────────────────────────────────────┐   │
│    │            MCP                       │   │
│    │    (Model Context Protocol)          │   │
│    └──────────────────────────────────────┘   │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│           底层服务 (Infrastructure)             │
│    File System | Database | API | Services     │
└─────────────────────────────────────────────────┘
```

**关键洞察**：
1. **MCP** 和 **A2A/ACP** 是**互补**关系，不是竞争关系
2. MCP 用于"一个模型访问多个工具"
3. A2A/ACP 用于"多个智能体相互通信"
4. 理想架构：**MCP + A2A 组合**

---

## 3. Claude Code & OpenCode 现状

### 3.1 Claude Code

**协议支持**：
- ✅ **MCP** - 原生支持，主要用于工具访问
- ✅ **Claude Skills** - Anthropic 的技能开放标准
- ❌ **ACP** - 不支持
- ❌ **A2A** - 尚不支持（但未来可能）

**架构特点**：
```typescript
Claude Code
    │
    ├─→ MCP Client → MCP Servers (Tools)
    │
    ├─→ Claude Skills (Project-specific instructions)
    │
    └─→ Anthropic API (LLM)
```

### 3.2 OpenCode

**协议支持**：
- ✅ **MCP** - 原生支持，配置简单
- ✅ **多模型集成** - 支持 75+ 模型提供商
- ✅ **ACP** - 部分支持（通过 ACP 适配器）
- ❌ **A2A** - 尚不支持

**架构特点**：
```
OpenCode
    │
    ├─→ MCP Client → MCP Servers (Tools)
    │
    ├─→ ACP Adapter (可选) → ACP Agents
    │
    └─→ Multi-Model Support
        ├─→ OpenAI
        ├─→ Anthropic
        ├─→ Local Models (Ollama, vLLM)
        └─→ 75+ Providers
```

### 3.3 集成挑战

**当前问题**：
1. **协议不统一**
   - Claude Code 使用 MCP
   - OpenCode 支持 MCP + ACP
   - 缺乏统一的智能体间通信标准

2. **状态同步困难**
   - 各工具独立运行
   - 缺乏共享状态机制
   - 难以实现复杂的多智能体协作

3. **发现机制缺失**
   - 无法自动发现可用的智能体
   - 需要手动配置每个连接

---

## 4. 协议选择建议

### 4.1 推荐方案：MCP + A2A 混合架构

基于分析，建议 Eigent 采用 **MCP + A2A 混合架构**：

```
┌─────────────────────────────────────────────────┐
│                Eigent Platform                   │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌────────────────────────────────────────┐    │
│  │      Multi-Agent Orchestration Layer   │    │
│  │           (基于 A2A 协议)              │    │
│  └──────────────┬─────────────────────────┘    │
│                 │                               │
│    ┌────────────┴────────────┬────────────┐    │
│    │                         │            │    │
│ ┌──▼────────┐  ┌───────────▼──┐  ┌──────▼──┐ │
│ │Claude Code│  │  OpenCode    │  │CodeX/Qwen│ │
│ │  Agent    │  │   Agent      │  │  Agent   │ │
│ └──┬────────┘  └───┬─────────┘  └─────┬────┘ │
│    │              │                │        │
│    └──────────────┴────────────────┘        │
│                   │                          │
│  ┌────────────────▼────────────────────────┐ │
│  │      MCP Tool Access Layer              │ │
│  │   (统一工具访问接口)                     │ │
│  └────────────────┬────────────────────────┘ │
│                   │                          │
│  ┌────────────────▼────────────────────────┐ │
│  │         MCP Servers                     │ │
│  │  File System | Database | API | Git    │ │
│  └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### 4.2 选择理由

#### 为什么选择 MCP 用于工具访问？

1. **成熟稳定**
   - 已有大量实现和案例
   - Anthropic 官方支持
   - 社区活跃

2. **Claude Code 原生支持**
   - 无需额外适配层
   - 集成简单
   - 性能最优

3. **专注工具调用**
   - 设计目标明确
   - 实现简单
   - 易于理解

#### 为什么选择 A2A 用于智能体通信？

1. **未来标准**
   - Google + IBM 联合支持
   - Linux Foundation 托管
   - 50+ 合作伙伴生态

2. **企业级特性**
   - 完整的状态管理
   - 高级安全性
   - 可扩展架构

3. **全面的功能**
   - 支持复杂协作
   - 动态发现
   - 长运行任务

#### 为什么不选择纯 ACP？

1. **维护风险**
   - 项目已合并到 A2A
   - 进入维护模式
   - 长期支持不确定

2. **生态限制**
   - 实现案例较少
   - 社区规模较小
   - 工具支持有限

### 4.3 过渡策略

**短期（2025 Q2-Q3）**：
- ✅ 使用 MCP 进行工具集成
- ✅ 实现 ACP 适配器（作为过渡方案）
- ✅ 关注 A2A 规范发布

**中期（2025 Q4-2026 Q1）**：
- ✅ 迁移到 A2A 协议
- ✅ 保持 MCP 用于工具访问
- ✅ 实现 A2A-MCP 桥接

**长期（2026 Q2+）**：
- ✅ 完整的 A2A + MCP 混合架构
- ✅ 支持动态智能体发现
- ✅ 企业级部署能力

---

## 5. 混合架构方案

### 5.1 架构设计

```typescript
// 混合协议管理器
interface HybridProtocolManager {
  // A2A 智能体通信
  a2a: {
    registerAgent(agent: A2AAgent): Promise<void>;
    discoverAgents(): Promise<A2AAgent[]>;
    sendMessage(to: string, message: A2AMessage): Promise<A2AMessage>;
    streamMessage(to: string, message: A2AMessage): AsyncStream<A2AMessage>;
  };

  // MCP 工具访问
  mcp: {
    connectServer(serverConfig: MCPServerConfig): Promise<MCPClient>;
    callTool(serverName: string, toolName: string, args: any): Promise<any>;
    listResources(serverName: string): Promise<Resource[]>;
  };

  // 协议桥接
  bridge: {
    a2aToMCP(a2aAgent: string): MCPAdapter;
    mcpToA2A(mcpTool: string): A2AAgent;
  };
}
```

### 5.2 实现示例

#### A2A 智能体包装器

```typescript
// a2a_agent_wrapper.ts
import { A2AClient } from '@a2a/sdk';

export class A2AAgentWrapper {
  private a2aClient: A2AClient;

  constructor(config: A2AConfig) {
    this.a2aClient = new A2AClient(config);
  }

  /**
   * 将 Claude Code 包装为 A2A 智能体
   */
  async wrapClaudeCode(claudeCodeConfig: ClaudeCodeConfig): Promise<void> {
    const agent = new A2AAgent({
      id: "claude-code",
      name: "Claude Code Agent",
      capabilities: [
        "code_generation",
        "code_review",
        "debugging",
        "file_operations"
      ],
      handler: async (message: A2AMessage) => {
        // 1. 将 A2A 消息转换为 Claude Code 格式
        const claudeMessage = this.a2aToClaude(message);

        // 2. 调用 Claude Code
        const response = await this.callClaudeCode(claudeMessage);

        // 3. 将响应转换回 A2A 格式
        return this.claudeToA2A(response);
      }
    });

    await this.a2aClient.register(agent);
  }

  /**
   * 将 OpenCode 包装为 A2A 智能体
   */
  async wrapOpenCode(openCodeConfig: OpenCodeConfig): Promise<void> {
    const agent = new A2AAgent({
      id: "opencode",
      name: "OpenCode Agent",
      capabilities: [
        "multi_model_generation",
        "code_completion",
        "refactoring"
      ],
      handler: async (message: A2AMessage) => {
        // 实现 OpenCode 调用逻辑
        const response = await this.callOpenCode(message);
        return response;
      }
    });

    await this.a2aClient.register(agent);
  }

  private async callClaudeCode(message: any): Promise<any> {
    // Claude Code 调用实现
  }

  private async callOpenCode(message: any): Promise<any> {
    // OpenCode 调用实现
  }

  private a2aToClaude(message: A2AMessage): any {
    // 消息格式转换
  }

  private claudeToA2A(response: any): A2AMessage {
    // 响应格式转换
  }
}
```

#### MCP-A2A 桥接

```typescript
// mcp_a2a_bridge.ts
export class MCPA2ABridge {
  private mcpClients: Map<string, MCPClient>;
  private a2aClient: A2AClient;

  constructor() {
    this.mcpClients = new Map();
    this.a2aClient = new A2AClient();
  }

  /**
   * 将 MCP 工具暴露为 A2A 智能体
   */
  async exposeMCPToolAsA2AAgent(
    serverName: string,
    toolName: string
  ): Promise<void> {
    const mcpClient = this.mcpClients.get(serverName);
    if (!mcpClient) {
      throw new Error(`MCP server ${serverName} not found`);
    }

    // 创建 A2A 智能体
    const agent = new A2AAgent({
      id: `mcp-${serverName}-${toolName}`,
      name: `${serverName}/${toolName}`,
      capabilities: ["tool_execution"],
      handler: async (message: A2AMessage) => {
        // 调用 MCP 工具
        const result = await mcpClient.callTool(toolName, message.data);
        return {
          id: this.generateId(),
          sender: `mcp-${serverName}-${toolName}`,
          receiver: message.sender,
          content: JSON.stringify(result),
          timestamp: new Date().toISOString()
        };
      }
    });

    await this.a2aClient.register(agent);
  }

  /**
   * 允许 A2A 智能体访问 MCP 工具
   */
  async enableMCPAccessForA2AAgent(
    agentId: string,
    serverName: string
  ): Promise<void> {
    const mcpClient = this.mcpClients.get(serverName);
    if (!mcpClient) {
      throw new Error(`MCP server ${serverName} not found`);
    }

    // 向 A2A 智能体注册 MCP 工具访问权限
    await this.a2aClient.grantCapability(agentId, {
      type: "mcp_access",
      server: serverName,
      actions: ["call_tool", "list_resources"]
    });
  }
}
```

### 5.3 工作流示例

```typescript
// multi_agent_workflow.ts
export class EigentWorkflow {
  private a2aManager: A2AAgentWrapper;
  private mcpBridge: MCPA2ABridge;

  constructor() {
    this.a2aManager = new A2AAgentWrapper();
    this.mcpBridge = new MCPA2ABridge();
  }

  /**
   * 代码审查工作流 - 使用 A2A 协调多个智能体
   */
  async codeReviewWorkflow(code: string): Promise<ReviewResult> {
    // 1. Claude Code 分析代码结构
    const analysis = await this.a2aManager.sendMessage(
      "claude-code",
      {
        content: `分析代码结构:\n${code}`,
        workflowId: "review-001"
      }
    );

    // 2. OpenCode 使用多模型进行安全检查
    const securityCheck = await this.a2aManager.sendMessage(
      "opencode",
      {
        content: `安全检查:\n${code}`,
        context: analysis.content,
        models: ["claude-3.5", "gpt-4", "deepseek-coder"]
      }
    );

    // 3. 通过 MCP 访问 Git 工具获取历史
    const gitHistory = await this.mcpBridge.callMCPTool(
      "github-server",
      "get_commit_history",
      { file: code }
    );

    // 4. Claude Code 综合结果
    const finalReport = await this.a2aManager.sendMessage(
      "claude-code",
      {
        content: `生成最终审查报告`,
        analysis: analysis.content,
        security: securityCheck.content,
        history: gitHistory
      }
    );

    return {
      analysis: analysis.content,
      security: securityCheck.content,
      history: gitHistory,
      report: finalReport.content
    };
  }
}
```

---

## 6. 实施路线图

### 6.1 阶段一：MCP 集成（立即开始）

**目标**：建立基于 MCP 的工具访问层

| 任务 | 优先级 | 预计时间 | 交付物 |
|-----|-------|---------|--------|
| MCP 客户端实现 | P0 | 1 周 | MCP Client |
| 文件系统服务器 | P0 | 1 周 | MCP FS Server |
| Git 服务器 | P1 | 1 周 | MCP Git Server |
| 数据库服务器 | P1 | 1 周 | MCP DB Server |
| 集成测试 | P1 | 1 周 | 测试套件 |

### 6.2 阶段二：A2A 准备（2025 Q3）

**目标**：为 A2A 集成做准备

| 任务 | 优先级 | 预计时间 | 交付物 |
|-----|-------|---------|--------|
| A2A 规范研究 | P0 | 持续 | 技术文档 |
| ACP 适配器（过渡） | P1 | 2 周 | ACP Adapter |
| 智能体包装器 | P1 | 2 周 | Agent Wrapper |
| 原型测试 | P1 | 1 周 | 原型系统 |

### 6.3 阶段三：A2A 集成（2025 Q4）

**目标**：实现 A2A 智能体通信

| 任务 | 优先级 | 预计时间 | 交付物 |
|-----|-------|---------|--------|
| A2A SDK 集成 | P0 | 2 周 | A2A Client |
| Claude Code 包装 | P0 | 1 周 | Claude Agent |
| OpenCode 包装 | P1 | 1 周 | OpenCode Agent |
| 状态管理 | P1 | 2 周 | State Manager |
| 桥接实现 | P1 | 2 周 | MCP-A2A Bridge |

### 6.4 阶段四：完善优化（2026 Q1+）

**目标**：优化和完善混合架构

| 任务 | 优先级 | 预计时间 | 交付物 |
|-----|-------|---------|--------|
| 性能优化 | P1 | 持续 | 优化报告 |
| 安全增强 | P0 | 2 周 | 安全方案 |
| 监控告警 | P1 | 2 周 | 监控系统 |
| 文档完善 | P1 | 2 周 | 完整文档 |

---

## 7. 关键决策点

### 7.1 何时使用 MCP？

✅ **使用 MCP 的场景**：
- 访问外部工具和资源
- 文件系统操作
- 数据库查询
- API 调用
- 单个 LLM 的上下文扩展

❌ **不使用 MCP 的场景**：
- 多智能体间直接通信
- 复杂的状态管理
- 长运行的协作任务

### 7.2 何时使用 A2A？

✅ **使用 A2A 的场景**：
- 多个智能体协作
- 跨组织的智能体通信
- 需要状态管理的任务
- 复杂的工作流编排
- 智能体发现和注册

❌ **不使用 A2A 的场景**：
- 简单的工具调用（用 MCP 更合适）
- 单个智能体的场景

### 7.3 何时使用 ACP？

⚠️ **ACP 使用建议**：
- 仅作为过渡方案
- 用于快速原型开发
- 等待 A2A 规范完全公开

❌ **不建议**：
- 新项目直接采用 ACP
- 长期依赖 ACP

---

## 8. 风险评估

### 8.1 技术风险

| 风险 | 影响 | 缓解措施 |
|-----|------|---------|
| A2A 规范未完全公开 | 高 | 先用 ACP 过渡，密切关注 A2A 发布 |
| 协议变更频繁 | 中 | 版本锁定，渐进式升级 |
| 性能问题 | 中 | 负载测试，优化关键路径 |
| 兼容性问题 | 中 | 充分测试，建立兼容性矩阵 |

### 8.2 业务风险

| 风险 | 影响 | 缓解措施 |
|-----|------|---------|
| Claude Code 不支持 A2A | 中 | 使用包装器，保持 MCP 集成 |
| OpenCode 协议变更 | 低 | 抽象层隔离，快速适配 |
| 社区支持不足 | 低 | 多协议支持，降低依赖 |

---

## 9. 总结与建议

### 9.1 核心建议

**推荐架构**：**MCP + A2A 混合架构**

```
工具层 → MCP
智能体层 → A2A
```

**实施策略**：
1. **短期**：全面采用 MCP 进行工具集成
2. **中期**：实现 ACP 作为 A2A 的过渡方案
3. **长期**：迁移到 A2A 进行智能体通信

### 9.2 关键要点

1. ✅ **MCP 和 A2A 是互补关系，不是竞争关系**
2. ✅ **A2A 是未来标准，值得投入**
3. ⚠️ **ACP 已合并到 A2A，谨慎使用**
4. ✅ **混合架构可以兼顾当前和未来需求**

### 9.3 行动清单

**立即行动**（本周）：
- [ ] 完成 MCP 服务器集成
- [ ] 创建 A2A 技术研究文档
- [ ] 评估 Claude Code 和 OpenCode 集成方案

**短期行动**（1-2 月）：
- [ ] 实现 MCP-A2A 桥接原型
- [ ] 开发智能体包装器
- [ ] 建立测试环境

**中期行动**（3-6 月）：
- [ ] 完整的 A2A 集成
- [ ] 生产环境部署
- [ ] 性能优化

---

## 参考资料

### 官方文档

1. **MCP (Model Context Protocol)**
   - [Anthropic - Introducing MCP](https://www.anthropic.com/news/model-context-protocol)
   - [MCP Specification](https://spec.modelcontextprotocol.io/)

2. **A2A (Agent-to-Agent Protocol)**
   - [A2A Protocol Official](https://a2a-protocol.org/latest/)
   - [Google - Announcing A2A](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
   - [A2A GitHub](https://github.com/a2aproject/A2A)

3. **ACP (Agent Communication Protocol)**
   - [ACP Official](https://agentcommunicationprotocol.dev/)
   - [IBM - ACP Overview](https://www.ibm.com/think/topics/agent-communication-protocol)

### 对比分析

1. **综合对比文章**
   - [An Unbiased Comparison of MCP, ACP, and A2A](https://medium.com/@sandibesen/an-unbiased-comparison-of-mcp-acp-and-a2a-protocols-0b45923a20f3)
   - [MCP, ACP, and A2A, Oh My!](https://camunda.com/blog/2025/05/mcp-acp-a2a-growing-world-inter-agent-communication/)
   - [Guide to AI Agent Protocols](https://getstream.io/blog/ai-agent-protocols/)

2. **技术深度分析**
   - [IBM TechXchange - Understanding MCP, ACP, and A2A](https://community.ibm.com/community/user/blogs/gyanendra-s-rathor/2025/06/26/understanding-mcp-acp-and-a2a)
   - [Comprehensive Analysis of Modern Protocols](https://inovaqo.com/2025/08/11/mcp-vs-a2a-vs-agp-vs-acp-a-comprehensive-analysis-of-modern-agent-communication-protocols/)

### Claude Code & OpenCode

1. **Claude Code**
   - [Claude Code Documentation](https://docs.anthropic.com/claude-code)
   - [Claude Skills Standard](https://docs.anthropic.com/claude-skills)

2. **OpenCode**
   - [OpenCode Technical Analysis](https://www.kevnu.com/zh/posts/opencode-in-depth-technical-analysis-of-open-source-ai-programming-agents)
   - [OpenCode vs Claude Code](https://help.apiyi.com/opencode-vs-claude-code-comparison.html)

### 中文资源

1. **协议对比（中文）**
   - [AI Agent 协议演进：从 MCP 到 ACP](https://juejin.cn/post/7577031454585929755)
   - [每位 AI 工程师都应了解的 A2A、MCP 与 ACP 协议](https://blog.csdn.net/m0_59235945/article/details/148184920)
   - [Agent2Agent 协议 - AI 智能体协作的新标准](https://blog.csdn.net/daiziguizhong/article/details/149594408)

2. **工具集成（中文）**
   - [从实现原理到 Claude Code、CodeX、OpenCode 实战](https://www.cnblogs.com/javastack/p/19531825)
   - [ACP：一个可能被低估的 Agent 接口协议](https://zhuanlan.zhihu.com/p/1994561622988055312)

---

**文档维护**: 本文档将随着协议发展和项目进展持续更新。

**贡献指南**: 欢迎提交问题和改进建议。

**最后更新**: 2025-02-06
