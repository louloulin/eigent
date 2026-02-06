# ACP 协议分析与 Claude Code 集成方案

## 📋 文档概述

**文档版本**: 1.0
**创建日期**: 2025-02-06
**最后更新**: 2025-02-06
**主题**: ACP (Agent Communication Protocol) 协议及其与 Claude Code 的集成

---

## 📚 目录

1. [ACP 协议概述](#1-acp-协议概述)
2. [ACP vs MCP vs A2A](#2-acp-vs-mcp-vs-a2a)
3. [ACP 技术规范](#3-acp-技术规范)
4. [Claude Code 现状分析](#4-claude-code-现状分析)
5. [ACP 与 Claude Code 集成方案](#5-acp-与-claude-code-集成方案)
6. [实施路线图](#6-实施路线图)
7. [代码示例](#7-代码示例)
8. [参考资料](#参考资料)

---

## 1. ACP 协议概述

### 1.1 什么是 ACP？

**ACP (Agent Communication Protocol)** 是一个开放标准，由 IBM 的 BeeAI 团队于 2025 年 3 月首次提出，旨在解决 AI 智能体之间的互操作性问题。

**核心定义**：
> ACP 是一个开放协议，用于智能体互操作性，解决连接 AI 智能体、应用程序和人类日益增长的挑战。

### 1.2 ACP 的设计目标

1. **框架无关性** - 支持任何技术栈和框架
2. **标准化通信** - 通过 RESTful API 实现统一的消息格式
3. **多模态支持** - 支持文本、图像、音频、视频等所有数据格式
4. **同步与异步** - 同时支持同步和异步通信模式
5. **流式交互** - 支持实时流式数据传输
6. **有状态与无状态** - 支持两种操作模式
7. **在线与离线发现** - 支持动态和静态智能体发现
8. **长运行任务** - 支持长时间运行的任务处理

### 1.3 ACP 的发展历程

| 时间 | 事件 |
|------|------|
| **2025年3月** | IBM Research 发布 ACP，用于驱动 BeeAI 平台 |
| **2025年4月** | ACP 首次代码提交 |
| **2025年5月** | IBM 正式发布 ACP（Google 发布 A2A 后一个月） |
| **2025年8月29日** | ACP 与 Google A2A 团队宣布合并，在 Linux Foundation 的 LF AI & Data 基金会下开发统一标准 |
| **2025年9月1日** | 官方确认合并，ACP 团队停止积极开发，将技术和专业知识贡献给 A2A |

### 1.4 ACP 的核心特性

#### REST-based Communication (基于 REST 的通信)

ACP 使用简单、定义明确的 REST 端点，符合标准 HTTP 模式。与需要 JSON-RPC 等专门通信方法的协议不同，ACP 利用了熟悉的 HTTP 约定，可以无缝集成到生产环境中。

```http
POST /agents/{agent_id}/invoke
Content-Type: application/json

{
  "messages": [...]
}
```

#### Support for All Message Types (支持所有消息类型)

ACP 使用 MimeTypes 进行内容识别，使其易于扩展以处理任何数据格式。无论是发送文本、图像、音频、视频还是自定义二进制格式，任何 mimetype 都可以开箱即用，无需修改协议。

#### No SDK Required (无需 SDK)

协议足够简单，可以使用标准 HTTP 工具如 curl、Postman 或浏览器请求。对于倾向于编程方式集成 ACP 的团队，提供官方 Python SDK 和 TypeScript SDK。

#### Offline Discovery (离线发现)

智能体可以通过将元数据直接嵌入到其分发包中来实现可发现性，即使在非活动状态也是如此。这使得在安全的、断开连接的或规模为零的环境中发现成为可能。

#### Async-first, Sync Supported (异步优先，支持同步)

主要用于异步通信以处理长时间运行的智能体任务，同时完全支持同步通信。

---

## 2. ACP vs MCP vs A2A

### 2.1 三大协议对比

| 特性 | **MCP** | **ACP** | **A2A** |
|------|---------|---------|---------|
| **全称** | Model Context Protocol | Agent Communication Protocol | Agent-to-Agent Protocol |
| **提出方** | Anthropic | IBM BeeAI | Google |
| **发布时间** | 2024年5月 | 2025年3月 | 2025年5月 |
| **核心焦点** | 单个模型与工具的交互 | 多智能体间通信 | 多智能体间通信 |
| **通信方式** | JSON-RPC | RESTful API | (待定) |
| **设计理念** | 一个人，多个工具 | 多个智能体协同工作 | 多个智能体对等协作 |
| **治理模式** | Anthropic 主导 | 开放治理（Linux Foundation） | Google 生态系统优化 |
| **SDK 要求** | 需要 SDK | 无需 SDK（可选） | (待定) |
| **流式支持** | 基本流式 | Delta 流式（细粒度） | (待定) |
| **内存共享** | 不支持 | 部分支持（开发中） | (待定) |
| **消息结构** | 任意 JSON schema | 标准化消息格式 | (待定) |

### 2.2 MCP 的局限性

为什么 MCP 不足以支持多智能体系统：

1. **流式限制**
   - MCP 支持基本流式（可能是完整消息的流式），但不支持更细粒度的"增量"风格
   - 增量流是即时发送的更新（如 token 和轨迹更新），由增量更新而非完整数据负载组成
   - 这种限制源于 MCP 创建时并非用于智能体风格的交互

2. **内存共享**
   - MCP 不支持跨服务器运行多个智能体同时维护共享内存
   - ACP 尚未完全支持此功能，但这是一个活跃的开发领域

3. **消息结构**
   - MCP 接受任何 JSON schema，但不定义消息体的结构
   - 这种灵活性使得互操作变得困难，尤其是对于必须解释不同消息格式的智能体

4. **协议复杂性**
   - MCP 使用 JSON-RPC，需要特定的 SDK 和运行时
   - ACP 的基于 REST 的设计内置异步/同步支持，更轻量级且易于集成

### 2.3 形象比喻

- **MCP**: 给一个人更好的工具（如计算器或参考书）来增强他们的性能
- **ACP**: 让人们形成团队，每个人（或智能体）在 AI 应用程序中协作贡献自己的能力

### 2.4 协议互补性

**MCP 和 ACP 可以互补**：

| 层次 | 协议 | 作用 |
|-----|------|------|
| **工具层** | MCP | 单个智能体访问工具、数据和资源 |
| **通信层** | ACP | 多个智能体之间的通信和协作 |
| **集成层** | ACP+MCP | 完整的多智能体协作系统 |

---

## 3. ACP 技术规范

### 3.1 REST API 端点

ACP 定义以下核心 REST 端点：

#### 智能体发现

```http
# 获取所有可用的智能体
GET /agents

# 获取特定智能体的元数据
GET /agents/{agent_id}

# 智能体健康检查
GET /agents/{agent_id}/health
```

#### 消息传递

```http
# 同步调用智能体
POST /agents/{agent_id}/invoke

# 异步调用智能体
POST /agents/{agent_id}/invoke_async

# 获取异步任务状态
GET /agents/{agent_id}/tasks/{task_id}

# 取消任务
DELETE /agents/{agent_id}/tasks/{task_id}
```

#### 流式通信

```http
# 服务器发送事件 (SSE) 流
GET /agents/{agent_id}/stream

# WebSocket 连接
WS /agents/{agent_id}/ws
```

### 3.2 消息格式

ACP 使用标准化的消息格式：

```typescript
interface ACPMessage {
  id: string;
  timestamp: string;
  sender: string;
  receiver: string;
  parts: MessagePart[];
  metadata?: Record<string, any>;
}

interface MessagePart {
  mimeType: string;
  content: string | binary;
  metadata?: Record<string, any>;
}
```

#### 示例消息

```json
{
  "id": "msg_123",
  "timestamp": "2025-02-06T10:00:00Z",
  "sender": "agent_a",
  "receiver": "agent_b",
  "parts": [
    {
      "mimeType": "text/plain",
      "content": "请分析这个代码库的结构"
    }
  ],
  "metadata": {
    "priority": "high",
    "requiresResponse": true
  }
}
```

### 3.3 多模态支持

ACP 通过 MimeTypes 支持多种数据格式：

```json
{
  "parts": [
    {
      "mimeType": "text/plain",
      "content": "这是文本内容"
    },
    {
      "mimeType": "image/png",
      "content": "base64_encoded_image_data"
    },
    {
      "mimeType": "audio/mp3",
      "content": "base64_encoded_audio_data"
    },
    {
      "mimeType": "application/json",
      "content": "{\"key\": \"value\"}"
    }
  ]
}
```

### 3.4 错误处理

ACP 定义标准错误格式：

```json
{
  "error": {
    "code": "AGENT_NOT_FOUND",
    "message": "The requested agent does not exist",
    "details": {
      "agent_id": "non_existent_agent"
    }
  }
}
```

---

## 4. Claude Code 现状分析

### 4.1 Claude Code 简介

**Claude Code** 是 Anthropic 推出的 AI 辅助编程工具，具有以下特点：

- ✅ 代码理解和生成
- ✅ 文件操作能力
- ✅ 终端命令执行
- ✅ 多轮对话
- ✅ 工具调用（Tool Use）
- ✅ MCP 协议支持

### 4.2 Claude Code 的通信方式

#### 当前架构

```
用户 → Claude Code → MCP Servers → 外部工具
         ↓
    Claude API
```

#### MCP 集成

Claude Code 已经支持 MCP 协议：

```typescript
// Claude Code 中的 MCP 客户端示例
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const client = new Client({
  name: "claude-code-client",
  version: "1.0.0"
});

const transport = new StdioClientTransport({
  command: "node",
  args: ["path/to/mcp/server.js"]
});

await client.connect(transport);
```

### 4.3 Claude Code 的局限性

1. **单智能体架构**
   - 当前主要是单个 Claude 实例
   - 多智能体协作需要自定义实现

2. **MCP 依赖**
   - MCP 主要用于工具访问
   - 不支持智能体间的直接通信

3. **缺乏标准化协议**
   - 与其他 AI 编程工具（如 Qwen Code、CodeX）的集成需要自定义适配器

---

## 5. ACP 与 Claude Code 集成方案

### 5.1 集成架构设计

#### 方案一：ACP 适配器层

```
┌─────────────────────────────────────────────────┐
│           Claude Code (现有)                    │
└───────────────────┬─────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────┐
│         ACP Adapter Layer (新增)                │
│  ┌─────────────────────────────────────────┐   │
│  │  • ACP Message → Claude Message         │   │
│  │  • Claude Message → ACP Message         │   │
│  │  • MCP Tool Call → ACP Agent Call       │   │
│  │  • Event Stream Translation             │   │
│  └─────────────────────────────────────────┘   │
└───────────────────┬─────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────┐
│              ACP Network                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Agent A  │  │ Agent B  │  │ Agent C  │      │
│  │ (Qwen)   │  │ (CodeX)  │  │ (Custom) │      │
│  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────┘
```

#### 方案二：MCP-ACP 桥接

```
┌─────────────────────────────────────────────────┐
│              Claude Code                        │
│  ┌─────────────────────────────────────────┐   │
│  │     MCP Client (现有)                    │   │
│  └──────────────┬──────────────────────────┘   │
└─────────────────┼───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│         MCP-ACP Bridge (新增)                    │
│  ┌─────────────────────────────────────────┐   │
│  │  • MCP Tool → ACP Agent                 │   │
│  │  • MCP Resource → ACP Agent Capability  │   │
│  │  • Protocol Translation                │   │
│  └─────────────────────────────────────────┘   │
└─────────────────┼───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│            ACP Network                           │
│  (其他 ACP 兼容的智能体)                          │
└─────────────────────────────────────────────────┘
```

### 5.2 技术实现方案

#### ACP 适配器实现

```typescript
// acp_adapter.ts
import { ACPClient } from '@acp/sdk';

export class ACPAdapter {
  private acpClient: ACPClient;

  constructor(config: ACPConfig) {
    this.acpClient = new ACPClient(config);
  }

  /**
   * 将 Claude 消息转换为 ACP 消息
   */
  claudeToACPMessages(claudeMessages: ClaudeMessage[]): ACPMessage[] {
    return claudeMessages.map(msg => ({
      id: this.generateId(),
      timestamp: new Date().toISOString(),
      sender: "claude-code",
      receiver: msg.targetAgent || "default",
      parts: [{
        mimeType: "text/plain",
        content: msg.content
      }],
      metadata: {
        source: "claude-code",
        conversationId: msg.conversationId
      }
    }));
  }

  /**
   * 将 ACP 消息转换为 Claude 消息
   */
  acpToClaudeMessages(acpMessages: ACPMessage[]): ClaudeMessage[] {
    return acpMessages.map(msg => ({
      role: msg.sender === "claude-code" ? "assistant" : "user",
      content: msg.parts[0].content,
      metadata: msg.metadata
    }));
  }

  /**
   * 调用 ACP 智能体
   */
  async invokeAgent(agentId: string, messages: ClaudeMessage[]): Promise<ClaudeMessage> {
    const acpMessages = this.claudeToACPMessages(messages);
    const acpResponse = await this.acpClient.invoke(agentId, acpMessages);
    return this.acpToClaudeMessages([acpResponse])[0];
  }

  /**
   * 流式调用 ACP 智能体
   */
  async* streamAgent(
    agentId: string,
    messages: ClaudeMessage[]
  ): AsyncGenerator<ClaudeMessageChunk> {
    const acpMessages = this.claudeToACPMessages(messages);
    const stream = this.acpClient.stream(agentId, acpMessages);

    for await (const chunk of stream) {
      yield {
        content: chunk.parts[0].content,
        delta: chunk.delta,
        metadata: chunk.metadata
      };
    }
  }

  private generateId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

#### MCP-ACP 桥接实现

```typescript
// mcp_acp_bridge.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { ACPClient } from '@acp/sdk';

export class MCPACPServer extends Server {
  private acpClient: ACPClient;

  constructor() {
    super({
      name: "mcp-acp-bridge",
      version: "1.0.0"
    }, {
      capabilities: {
        tools: {},
        resources: {}
      }
    });

    this.acpClient = new ACPClient();
    this.setupHandlers();
  }

  private setupHandlers() {
    // 将 MCP 工具调用转换为 ACP 智能体调用
    this.setRequestHandler(CallToolRequestSchema, async (request) => {
      const { name, arguments: args } = request.params;

      // 解析智能体名称
      const agentId = this.extractAgentId(name);

      // 调用 ACP 智能体
      const acpMessage: ACPMessage = {
        id: this.generateId(),
        timestamp: new Date().toISOString(),
        sender: "mcp-acp-bridge",
        receiver: agentId,
        parts: [{
          mimeType: "application/json",
          content: JSON.stringify(args)
        }]
      };

      const response = await this.acpClient.invoke(agentId, acpMessage);

      return {
        content: [{
          type: "text",
          text: JSON.stringify(response)
        }]
      };
    });

    // 将 MCP 资源请求转换为 ACP 智能体能力查询
    this.setRequestHandler(ListResourcesRequestSchema, async () => {
      const agents = await this.acpClient.listAgents();

      return {
        resources: agents.map(agent => ({
          uri: `acp://${agent.id}`,
          name: agent.name,
          description: agent.description,
          mimeType: "application/json"
        }))
      };
    });
  }

  private extractAgentId(toolName: string): string {
    // acp_agent_developer → developer
    const match = toolName.match(/^acp_agent_(.+)$/);
    return match ? match[1] : toolName;
  }

  private generateId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### 5.3 集成后的工作流程

#### 多智能体协作场景

```typescript
// multi_agent_workflow.ts
import { ACPAdapter } from './acp_adapter';

export class MultiAgentWorkflow {
  private acpAdapter: ACPAdapter;

  constructor() {
    this.acpAdapter = new ACPAdapter({
      baseUrl: process.env.ACP_SERVER_URL,
      timeout: 30000
    });
  }

  /**
   * 代码审查工作流
   */
  async codeReviewWorkflow(code: string): Promise<ReviewResult> {
    // Step 1: 使用 Claude Code 进行初步分析
    const initialAnalysis = await this.acpAdapter.invokeAgent(
      "claude-code",
      [{ content: `分析这段代码的结构和质量:\n${code}` }]
    );

    // Step 2: 使用 Qwen Code 进行安全检查
    const securityCheck = await this.acpAdapter.invokeAgent(
      "qwen-code",
      [{ content: `检查这段代码的安全问题:\n${code}` }]
    );

    // Step 3: 使用专业测试 Agent 生成测试
    const testGeneration = await this.acpAdapter.invokeAgent(
      "test-agent",
      [{
        content: `基于以下分析和代码生成单元测试:\n` +
                 `分析: ${initialAnalysis.content}\n` +
                 `安全问题: ${securityCheck.content}\n` +
                 `代码: ${code}`
      }]
    );

    // Step 4: Claude Code 综合结果
    const finalReport = await this.acpAdapter.invokeAgent(
      "claude-code",
      [{
        content: `综合以下信息生成最终代码审查报告:\n` +
                 `1. 初步分析: ${initialAnalysis.content}\n` +
                 `2. 安全检查: ${securityCheck.content}\n` +
                 `3. 测试建议: ${testGeneration.content}`
      }]
    );

    return {
      analysis: initialAnalysis.content,
      securityIssues: securityCheck.content,
      tests: testGeneration.content,
      report: finalReport.content
    };
  }
}
```

---

## 6. 实施路线图

### 6.1 阶段一：基础设施 (Q1 2025)

**目标**: 建立 ACP 集成的基础架构

| 任务 | 优先级 | 预计时间 | 负责人 |
|-----|-------|---------|--------|
| ACP SDK 集成 | P0 | 1 周 | Backend Team |
| ACP 适配器开发 | P0 | 2 周 | Full Stack |
| MCP-ACP 桥接原型 | P1 | 1 周 | Backend Team |
| 单元测试 | P1 | 1 周 | QA Team |

### 6.2 阶段二：核心功能 (Q2 2025)

**目标**: 实现基本的多智能体通信

| 任务 | 优先级 | 预计时间 | 负责人 |
|-----|-------|---------|--------|
| 消息格式转换 | P0 | 2 周 | Full Stack |
| 流式通信支持 | P0 | 2 周 | Backend Team |
| 错误处理机制 | P1 | 1 周 | Full Stack |
| 集成测试 | P1 | 1 周 | QA Team |

### 6.3 阶段三：高级特性 (Q3 2025)

**目标**: 实现高级协作功能

| 任务 | 优先级 | 预计时间 | 负责人 |
|-----|-------|---------|--------|
| 智能体发现机制 | P1 | 2 周 | Backend Team |
| 任务队列管理 | P1 | 2 周 | Full Stack |
| 状态同步 | P2 | 2 周 | Backend Team |
| 性能优化 | P2 | 持续 | All Teams |

### 6.4 阶段四：生态集成 (Q4 2025)

**目标**: 与其他智能体平台集成

| 任务 | 优先级 | 预计时间 | 负责人 |
|-----|-------|---------|--------|
| Qwen Code 集成 | P1 | 2 周 | Integration Team |
| CodeX 集成 | P1 | 2 周 | Integration Team |
| 自定义智能体支持 | P2 | 3 周 | Backend Team |
| 文档和示例 | P1 | 2 周 | Docs Team |

---

## 7. 代码示例

### 7.1 完整的 ACP 智能体实现

```typescript
// claude_code_acp_agent.ts
import { ACPAgent, ACPMessage, ACPContext } from '@acp/sdk';

export class ClaudeCodeACPAgent extends ACPAgent {
  private claudeClient: ClaudeClient;

  constructor(config: AgentConfig) {
    super({
      id: "claude-code",
      name: "Claude Code Agent",
      version: "1.0.0",
      capabilities: [
        "code_generation",
        "code_review",
        "code_explanation",
        "debugging"
      ]
    });

    this.claudeClient = new ClaudeClient(config.apiKey);
  }

  /**
   * 处理传入的 ACP 消息
   */
  async handleMessage(message: ACPMessage, context: ACPContext): Promise<ACPMessage> {
    const startTime = Date.now();

    try {
      // 1. 解析消息内容
      const request = this.parseRequest(message);

      // 2. 调用 Claude API
      const claudeResponse = await this.claudeClient.messages.create({
        model: "claude-3-5-sonnet-20241022",
        max_tokens: 4096,
        messages: [
          {
            role: "user",
            content: request.content
          }
        ],
        tools: this.getACPAvailableTools()
      });

      // 3. 处理工具调用
      if (claudeResponse.stop_reason === "tool_use") {
        const toolResults = await this.executeTools(
          claudeResponse.content.filter(block => block.type === "tool_use")
        );

        // 继续对话以获取最终响应
        const finalResponse = await this.claudeClient.messages.create({
          model: "claude-3-5-sonnet-20241022",
          max_tokens: 4096,
          messages: [
            { role: "user", content: request.content },
            ...claudeResponse.content,
            ...toolResults
          ]
        });

        return this.formatResponse(finalResponse, context);
      }

      // 4. 格式化 ACP 响应
      return this.formatResponse(claudeResponse, context);

    } catch (error) {
      return this.formatError(error, context);
    }
  }

  /**
   * 流式处理消息
   */
  async* streamMessage(
    message: ACPMessage,
    context: ACPContext
  ): AsyncGenerator<ACPMessageChunk> {
    try {
      const request = this.parseRequest(message);

      const stream = await this.claudeClient.messages.create({
        model: "claude-3-5-sonnet-20241022",
        max_tokens: 4096,
        messages: [{ role: "user", content: request.content }],
        stream: true
      });

      for await (const chunk of stream) {
        if (chunk.type === "content_block_delta") {
          yield {
            type: "delta",
            content: chunk.delta.text,
            timestamp: new Date().toISOString()
          };
        }
      }
    } catch (error) {
      yield {
        type: "error",
        error: error.message,
        timestamp: new Date().toISOString()
      };
    }
  }

  /**
   * 获取 ACP 可用的工具
   */
  private getACPAvailableTools() {
    return [
      {
        name: "read_file",
        description: "Read the contents of a file",
        input_schema: {
          type: "object",
          properties: {
            path: {
              type: "string",
              description: "Path to the file to read"
            }
          },
          required: ["path"]
        }
      },
      {
        name: "write_file",
        description: "Write content to a file",
        input_schema: {
          type: "object",
          properties: {
            path: {
              type: "string",
              description: "Path to the file to write"
            },
            content: {
              type: "string",
              description: "Content to write"
            }
          },
          required: ["path", "content"]
        }
      },
      {
        name: "execute_command",
        description: "Execute a shell command",
        input_schema: {
          type: "object",
          properties: {
            command: {
              type: "string",
              description: "Command to execute"
            }
          },
          required: ["command"]
        }
      }
    ];
  }

  /**
   * 执行工具调用
   */
  private async executeTools(toolUses: any[]): Promise<any[]> {
    const results = [];

    for (const toolUse of toolUses) {
      try {
        let result;

        switch (toolUse.name) {
          case "read_file":
            result = await this.readFile(toolUse.input.path);
            break;
          case "write_file":
            result = await this.writeFile(toolUse.input.path, toolUse.input.content);
            break;
          case "execute_command":
            result = await this.executeCommand(toolUse.input.command);
            break;
          default:
            result = { error: `Unknown tool: ${toolUse.name}` };
        }

        results.push({
          type: "tool_result",
          tool_use_id: toolUse.id,
          content: JSON.stringify(result)
        });
      } catch (error) {
        results.push({
          type: "tool_result",
          tool_use_id: toolUse.id,
          content: JSON.stringify({ error: error.message }),
          is_error: true
        });
      }
    }

    return results;
  }

  private parseRequest(message: ACPMessage): Request {
    const textPart = message.parts.find(p => p.mimeType.startsWith("text/"));
    return {
      content: textPart?.content || "",
      metadata: message.metadata
    };
  }

  private formatResponse(claudeResponse: any, context: ACPContext): ACPMessage {
    const textContent = claudeResponse.content
      .filter(block => block.type === "text")
      .map(block => block.text)
      .join("\n");

    return {
      id: this.generateId(),
      timestamp: new Date().toISOString(),
      sender: "claude-code",
      receiver: context.sender,
      parts: [{
        mimeType: "text/plain",
        content: textContent
      }],
      metadata: {
        model: claudeResponse.model,
        usage: claudeResponse.usage
      }
    };
  }

  private formatError(error: any, context: ACPContext): ACPMessage {
    return {
      id: this.generateId(),
      timestamp: new Date().toISOString(),
      sender: "claude-code",
      receiver: context.sender,
      parts: [{
        mimeType: "text/plain",
        content: `Error: ${error.message}`
      }],
      metadata: {
        error: true,
        errorCode: error.code || "UNKNOWN_ERROR"
      }
    };
  }

  private generateId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // 工具方法实现
  private async readFile(path: string): Promise<string> {
    // 实现
    return "";
  }

  private async writeFile(path: string, content: string): Promise<void> {
    // 实现
  }

  private async executeCommand(command: string): Promise<string> {
    // 实现
    return "";
  }
}
```

### 7.2 使用 ACP 的 Claude Code 客户端

```typescript
// acp_claude_code_client.ts
import { ACPClient } from '@acp/sdk';
import { ClaudeCodeACPAgent } from './claude_code_acp_agent';

export class ACPClaudeCodeClient {
  private acpClient: ACPClient;
  private agent: ClaudeCodeACPAgent;

  constructor(config: ClientConfig) {
    // 初始化 ACP 客户端
    this.acpClient = new ACPClient({
      serverUrl: config.acpServerUrl,
      timeout: config.timeout || 30000
    });

    // 创建并注册 Claude Code 智能体
    this.agent = new ClaudeCodeACPAgent({
      apiKey: config.anthropicApiKey
    });
  }

  /**
   * 启动智能体服务
   */
  async start(): Promise<void> {
    // 注册到 ACP 网络
    await this.acpClient.registerAgent(this.agent);

    // 启动 REST API 服务器
    await this.agent.start({
      port: 8080,
      endpoints: {
        invoke: "/agents/claude-code/invoke",
        stream: "/agents/claude-code/stream",
        health: "/agents/claude-code/health"
      }
    });

    console.log("Claude Code ACP Agent started on port 8080");
  }

  /**
   * 发现其他智能体
   */
  async discoverAgents(): Promise<AgentInfo[]> {
    const agents = await this.acpClient.listAgents();
    return agents.map(agent => ({
      id: agent.id,
      name: agent.name,
      capabilities: agent.capabilities,
      status: agent.status
    }));
  }

  /**
   * 调用其他智能体
   */
  async callAgent(agentId: string, message: string): Promise<string> {
    const response = await this.acpClient.invoke(agentId, {
      id: this.generateId(),
      timestamp: new Date().toISOString(),
      sender: "claude-code",
      receiver: agentId,
      parts: [{
        mimeType: "text/plain",
        content: message
      }]
    });

    return response.parts[0].content;
  }

  /**
   * 多智能体协作示例
   */
  async multiAgentCollaboration(task: string): Promise<CollaborationResult> {
    // 1. 发现可用的智能体
    const agents = await this.discoverAgents();

    // 2. 选择合适的智能体
    const codeAgents = agents.filter(a =>
      a.capabilities.includes("code_generation")
    );

    // 3. 并行调用多个智能体
    const results = await Promise.all(
      codeAgents.map(agent =>
        this.callAgent(agent.id, task)
      )
    );

    // 4. 聚合结果
    return {
      task,
      results,
      timestamp: new Date().toISOString()
    };
  }

  private generateId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// 使用示例
async function main() {
  const client = new ACPClaudeCodeClient({
    acpServerUrl: process.env.ACP_SERVER_URL || "http://localhost:8000",
    anthropicApiKey: process.env.ANTHROPIC_API_KEY
  });

  await client.start();

  // 多智能体协作
  const result = await client.multiAgentCollaboration(
    "实现一个快速排序算法，并进行代码审查和测试"
  );

  console.log("协作结果:", result);
}
```

---

## 8. 参考资料

### 官方文档

1. **ACP 官方网站**
   - [Agent Communication Protocol - Welcome](https://agentcommunicationprotocol.dev/introduction/welcome)

2. **IBM ACP 文档**
   - [IBM - Agent Communication Protocol (ACP)](https://www.ibm.com/think/topics/agent-communication-protocol)

3. **DeepLearning.AI ACP 课程**
   - [ACP: Agent Communication Protocol](https://learn.deeplearning.ai/courses/acp-agent-communication-protocol/information)

### 技术文章

1. **Medium - ACP 介绍**
   - [Introducing the Agent Communication Protocol (ACP)](https://medium.com/mitb-for-all/introducing-the-agent-communication-protocol-acp-abd882114139)

2. **Towards Data Science**
   - [The Future of AI Agent Communication with ACP](https://towardsdatascience.com/the-future-of-ai-agent-communication-with-acp/)

3. **IBM 教程（中文）**
   - [使用 ACP 实现 AI 智能体互操作性：构建多智能体工作流](https://www.ibm.com/cn-zh/think/tutorials/acp-ai-agent-interoperability-building-multi-agent-workflows)

### 社区资源

1. **GitHub**
   - [i-am-bee/acp](https://github.com/i-am-bee/acp) - ACP 实现

2. **Open Agent School**
   - [ACP Concepts](https://www.openagentschool.org/concepts/acp)

3. **IBM Think**
   - [ACP 与 A2A 合并公告](https://www.ibm.com/think/topics/agent-communication-protocol)

### 相关协议

1. **MCP (Model Context Protocol)**
   - [Anthropic - Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol)

2. **A2A (Agent-to-Agent Protocol)**
   - [Google - Announcing the Agent2Agent Protocol](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)

3. **ACP-MCP Bridge**
   - [LobeHub - ACP-MCP-Server](https://lobehub.com/mcp/gongrzhe-acp-mcp-server)

### 中文资源

1. **掘金**
   - [AI Agent 协议演进：从 MCP 到 ACP 的架构对比与未来展望](https://juejin.cn/post/7577031454585929755)

2. **腾讯云开发者**
   - [读懂 MCP 协议：AI Agent 开发者的必备通信语言](https://cloud.tencent.com/developer/article/2535686)

3. **CSDN**
   - [每位 AI 工程师都应了解的 A2A、MCP 与 ACP 协议](https://blog.csdn.net/m0_59235945/article/details/148184920)

---

## 📝 附录

### A. 术语表

| 术语 | 全称 | 定义 |
|-----|------|------|
| **ACP** | Agent Communication Protocol | 智能体通信协议，由 IBM 提出的多智能体通信标准 |
| **MCP** | Model Context Protocol | 模型上下文协议，由 Anthropic 提出的工具访问标准 |
| **A2A** | Agent-to-Agent Protocol | 智能体对智能体协议，由 Google 提出的通信协议 |
| **BeeAI** | - | IBM 的开源 AI 智能体框架 |
| **REST** | Representational State Transfer | 表述性状态传递，一种 API 设计风格 |
| **SSE** | Server-Sent Events | 服务器发送事件，一种流式通信技术 |

### B. ACP 消息示例

#### 基本文本消息

```json
{
  "id": "msg_001",
  "timestamp": "2025-02-06T10:00:00Z",
  "sender": "agent_a",
  "receiver": "agent_b",
  "parts": [
    {
      "mimeType": "text/plain",
      "content": "你好，请帮我分析这段代码"
    }
  ]
}
```

#### 多模态消息

```json
{
  "id": "msg_002",
  "timestamp": "2025-02-06T10:05:00Z",
  "sender": "user",
  "receiver": "claude-code",
  "parts": [
    {
      "mimeType": "text/plain",
      "content": "这个界面设计有什么问题？"
    },
    {
      "mimeType": "image/png",
      "content": "iVBORw0KGgoAAAANSUhEUgAAAAUA..."
    },
    {
      "mimeType": "application/json",
      "content": "{\"context\": \"ui_review\", \"priority\": \"high\"}"
    }
  ],
  "metadata": {
    "conversationId": "conv_123",
    "requiresResponse": true
  }
}
```

#### 流式消息

```json
{
  "id": "msg_003",
  "timestamp": "2025-02-06T10:10:00Z",
  "sender": "claude-code",
  "receiver": "user",
  "parts": [
    {
      "mimeType": "text/plain",
      "content": "让我来分析",
      "delta": true
    }
  ],
  "metadata": {
    "streamId": "stream_001",
    "sequence": 1
  }
}
```

### C. 快速开始指南

#### 安装 ACP SDK

```bash
# Python
pip install acp-sdk

# TypeScript/JavaScript
npm install @acp/sdk
```

#### 创建简单的 ACP 智能体

```python
# Python 示例
from acp_sdk import Server, Message
from acp_sdk.models import MessagePart

server = Server()

@server.agent()
async def my_agent(messages: list[Message]):
    """简单的 ACP 智能体"""
    for message in messages:
        # 处理消息
        response = process_message(message)

        # 返回响应
        yield Message(parts=[
            MessagePart(content=response)
        ])

if __name__ == "__main__":
    server.run()
```

#### 启动 ACP 服务器

```bash
# 使用 Python
python my_agent.py --port 8080

# 使用 TypeScript
node dist/my_agent.js --port 8080
```

#### 测试 ACP 智能体

```bash
# 使用 curl
curl -X POST http://localhost:8080/agents/my_agent/invoke \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "parts": [{"content": "你好"}]
      }
    ]
  }'
```

---

## 🎯 总结

### ACP 的价值

1. **标准化** - 提供统一的多智能体通信标准
2. **互操作性** - 不同框架和平台的智能体可以无缝协作
3. **可扩展性** - 支持自定义消息类型和协议扩展
4. **简单性** - 基于 REST，无需复杂的 SDK

### 与 Claude Code 集成的优势

1. **多智能体协作** - Claude Code 可以与其他 AI 编程工具协同工作
2. **能力扩展** - 通过 ACP 网络访问更多专业智能体
3. **生态系统** - 参与构建开放的多智能体生态系统
4. **未来兼容** - 为 A2A 等未来协议的迁移做好准备

### 下一步行动

1. ✅ 学习 ACP 协议规范
2. ✅ 评估 Claude Code 的 ACP 集成需求
3. ✅ 开发 ACP 适配器原型
4. ✅ 测试多智能体协作场景
5. ✅ 参与 ACP/A2A 社区讨论

---

**文档维护**: 本文档将随着 ACP 和 Claude Code 的发展持续更新。

**贡献指南**: 欢迎提交问题和改进建议到 Eigent 项目的 GitHub 仓库。

**联系方式**: 如有疑问，请通过 GitHub Issues 或 Discord 与团队联系。
