# Eigent 技术架构文档

## 项目概述

**Eigent** 是一个开源的多智能体协作桌面应用程序，基于 CAMEL-AI 框架构建，让用户能够构建、管理和部署自定义 AI 工作流，将复杂的工作流程转变为自动化任务。

### 核心特性

- **多智能体编排**：基于 CAMEL-AI 的 Workforce 系统，实现 Agent 间并行协作
- **MCP 协议集成**：通过 Model Context Protocol 无缝集成第三方工具
- **本地优先**：支持完全本地部署，保障数据隐私
- **实时通信**：基于 SSE 的流式响应，提供即时反馈
- **可扩展架构**：Agent Factory + Toolkit 抽象，易于扩展

---

## 1. 系统架构概览

### 1.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         Eigent Desktop                           │
│                    (Electron + React + TypeScript)               │
└───────────────────────────────┬─────────────────────────────────┘
                                │ SSE / HTTP API
┌───────────────────────────────▼─────────────────────────────────┐
│                       Backend (FastAPI)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Chat Service │  │ Task Service │  │ Model Config │          │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘          │
│         │                  │                                     │
│  ┌──────▼──────────────────▼──────┐                             │
│  │      Shared Task Channel       │                             │
│  └──────┬──────────────────┬──────┘                             │
│         │                  │                                     │
│  ┌──────▼─────────┐  ┌─────▼──────────┐                         │
│  │   Workforce    │  │  Agent Factory │                         │
│  │  (CAMEL-AI)    │  │    System      │                         │
│  └──────┬─────────┘  └─────┬──────────┘                         │
│         │                  │                                     │
│  ┌──────▼──────────────────▼──────┐                             │
│  │      Toolkit System            │                             │
│  │  (30+ Specialized Toolkits)    │                             │
│  └────────────────────────────────┘                             │
└───────────────────────────────┬─────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
┌───────▼────────┐    ┌────────▼────────┐    ┌────────▼────────┐
│  Local Models  │    │  Cloud Models   │    │  MCP Servers    │
│  (vLLM/Ollama) │    │  (OpenAI/etc)   │    │  (External)     │
└────────────────┘    └─────────────────┘    └─────────────────┘
```

### 1.2 项目结构

```
eigent/
├── backend/                    # Python 后端服务
│   ├── app/
│   │   ├── agent/             # Agent 系统
│   │   │   ├── factory/       # Agent 工厂
│   │   │   ├── listen_chat_agent.py  # 核心 Agent 类
│   │   │   ├── tools.py       # 工具集成
│   │   │   └── prompt.py      # 系统提示词
│   │   ├── controller/        # API 控制器
│   │   ├── service/           # 业务逻辑层
│   │   ├── model/             # 数据模型
│   │   └── utils/             # 工具模块
│   │       ├── toolkit/       # 30+ 工具包
│   │       ├── telemetry/     # 监控遥测
│   │       └── server/        # 服务器工具
│   ├── main.py                # 应用入口
│   └── pyproject.toml         # Python 依赖
├── src/                        # React 前端
│   ├── components/            # UI 组件
│   │   ├── WorkFlow/         # 工作流编辑器
│   │   ├── ChatBox/          # 聊天界面
│   │   ├── ui/               # 基础 UI 组件
│   │   └── [AgentWorkspaces] # Agent 工作区
│   ├── store/                # Zustand 状态管理
│   ├── pages/                # 页面组件
│   ├── hooks/                # 自定义 Hooks
│   ├── api/                  # API 客户端
│   ├── lib/                  # 工具库
│   └── types/                # TypeScript 类型
├── electron/                  # Electron 主进程
├── docs/                      # 文档
├── package.json              # 前端依赖
└── vite.config.ts           # Vite 配置
```

---

## 2. 后端架构

### 2.1 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **Python** | 3.11+ | 核心语言 |
| **FastAPI** | 0.115.12 | Web 框架 |
| **CAMEL-AI** | 0.2.85a0 | Agent 框架 |
| **Uvicorn** | Latest | ASGI 服务器 |
| **PostgreSQL** | Latest | 数据持久化 |
| **Qdrant** | Latest | 向量存储 |
| **Redis** | Latest | 缓存（可选） |

### 2.2 应用启动流程

```python
# backend/main.py
def main():
    # 1. 创建 FastAPI 应用
    app = create_app()

    # 2. 注册路由
    register_controllers(app)

    # 3. 配置中间件
    setup_middleware(app)

    # 4. 启动 Uvicorn 服务器
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 2.3 路由架构

```
/api
├── /health          # 健康检查
├── /chat            # 聊天相关
│   ├── /stream     # SSE 流式响应
│   └── /history    # 聊天历史
├── /task            # 任务管理
│   ├── /start      # 启动任务
│   ├── /stop       # 停止任务
│   └── /update     # 更新任务
├── /model           # 模型配置
│   ├── /list       # 模型列表
│   └── /config     # 模型配置
└── /tool            # 工具管理
    ├── /list       # 工具列表
    └── /mcp        # MCP 服务器
```

---

## 3. Agent 系统

### 3.1 Agent 架构

```
                    ┌─────────────────┐
                    │  Agent Factory  │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │            │           │           │
┌───────▼──────┐ ┌───▼────┐ ┌──▼─────┐ ┌──▼────────┐
│  Developer   │ │ Browser│ │Document│ │MultiModal │
│   Agent      │ │ Agent  │ │ Agent  │ │  Agent    │
└───────┬──────┘ └───┬────┘ └──┬─────┘ └──┬────────┘
        │            │          │           │
        └────────────┼──────────┼───────────┘
                     │          │
              ┌──────▼──────────▼──────┐
              │   ListenChatAgent      │
              │  (继承 CAMEL ChatAgent) │
              └──────┬─────────────────┘
                     │
         ┌───────────┼───────────┐
         │           │           │
    ┌────▼────┐ ┌───▼────┐ ┌───▼────┐
    │ Toolkit │ │ Memory │ │  LLM   │
    │  System │ │        │ │ Backend│
    └─────────┘ └────────┘ └────────┘
```

### 3.2 核心 Agent 类

**ListenChatAgent** (`backend/app/agent/listen_chat_agent.py`)

```python
class ListenChatAgent(ChatAgent):
    """扩展自 CAMEL 的 ChatAgent，添加：
    - Agent 生命周期事件（激活/停用）
    - Toolkit 执行监听
    - 流式响应处理
    - Token 使用追踪
    """

    def __init__(self, api_task_id, agent_name, ...):
        super().__init__(...)
        self.api_task_id = api_task_id
        self.agent_name = agent_name

    def step(self, input_message):
        # 1. 发送激活事件
        self._send_agent_activate()

        # 2. 执行 Agent 逻辑
        response = super().step(input_message)

        # 3. 发送停用事件
        self._send_agent_deactivate()

        return response

    def _execute_tool(self, tool_call_request):
        # 执行工具并发送工具事件
        pass
```

### 3.3 Agent 工厂模式

**工厂函数** (`backend/app/agent/agent_model.py`)

```python
async def agent_model(
    agent_name: str,
    system_message: str,
    options: Chat,
    tools: list,
    tool_names: list
) -> ListenChatAgent:
    """创建配置好的 Agent 实例"""

    # 1. 准备模型后端
    model_backend = get_model_backend(options)

    # 2. 创建 Agent
    agent = ListenChatAgent(
        system_message=system_message,
        model=model_backend,
        tools=tools,
        ...
    )

    # 3. 设置项目上下文
    set_project_context(options)

    return agent
```

**专用 Agent 工厂**

| 工厂文件 | Agent 类型 | 核心能力 |
|---------|-----------|---------|
| `factory/developer.py` | DeveloperAgent | 代码编写、终端操作、Web 部署 |
| `factory/browser.py` | BrowserAgent | 网页浏览、搜索、内容提取 |
| `factory/document.py` | DocumentAgent | 文档处理、演示文稿 |
| `factory/multi_modal.py` | MultiModalAgent | 图像、音频、视频处理 |
| `factory/social_media.py` | SocialMediaAgent | 社交媒体操作 |
| `factory/mcp.py` | MCPAgent | MCP 工具集成 |

### 3.4 Agent 示例：Developer Agent

```python
# backend/app/agent/factory/developer.py
async def developer_agent(options: Chat) -> ListenChatAgent:
    # 1. 准备工作目录
    working_directory = get_working_directory(options)

    # 2. 创建工具集成
    message_integration = ToolkitMessageIntegration(...)

    # 3. 装备 Toolkits
    tools = [
        *HumanToolkit.get_can_use_tools(...),
        *terminal_toolkit.get_tools(),
        *note_toolkit.get_tools(),
        *web_deploy_toolkit.get_tools(),
    ]

    # 4. 创建系统消息
    system_message = DEVELOPER_SYS_PROMPT.format(
        working_directory=working_directory,
        ...
    )

    # 5. 返回配置好的 Agent
    return await agent_model(
        agent_name="developer_agent",
        system_message=system_message,
        options=options,
        tools=tools,
        tool_names=["TerminalToolkit", "NoteTakingToolkit", ...]
    )
```

---

## 4. Toolkit 系统

### 4.1 Toolkit 架构

```
                    ┌──────────────────┐
                    │ AbstractToolkit │
                    │   (抽象基类)     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼──────┐    ┌────────▼────────┐   ┌─────▼────────┐
│ Development  │    │   Web/Network   │   │  Media/IO    │
│   Toolkits   │    │    Toolkits     │   │   Toolkits   │
├──────────────┤    ├─────────────────┤   ├──────────────┤
│ Terminal     │    │ HybridBrowser   │   │ ImageAnalysis│
│ WebDeploy    │    │ Search          │   │ AudioAnalysis│
│ CodeExecution│    │ GitHub          │   │ VideoAnalysis│
│ FileWrite    │    │ Craw4ai         │   │ OpenAIImage  │
└──────────────┘    └─────────────────┘   └──────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Integration    │
                    │    Layer        │
                    └─────────────────┘
```

### 4.2 Toolkit 分类

#### 开发工具类
- **TerminalToolkit**: 跨平台终端命令执行
- **WebDeployToolkit**: 本地 Web 应用部署
- **CodeExecutionToolkit**: 代码执行环境
- **FileWriteToolkit**: 文件创建与写入

#### 网络工具类
- **HybridBrowserToolkit**: 状态化浏览器控制
- **SearchToolkit**: 多搜索引擎集成
- **GitHubToolkit**: GitHub 仓库操作
- **Craw4aiToolkit**: 网页爬虫

#### 文档工具类
- **PPTXToolkit**: PowerPoint 演示文稿
- **ExcelToolkit**: Excel 表格操作
- **MarkItDownToolkit**: 多格式转 Markdown
- **NoteTakingToolkit**: 笔记管理

#### 媒体处理类
- **ImageAnalysisToolkit**: 图像内容理解
- **AudioAnalysisToolkit**: 音频转录与分析
- **VideoAnalysisToolkit**: 视频内容分析
- **VideoDownloaderToolkit**: 视频下载
- **OpenAIImageToolkit**: DALL-E 图像生成

#### 云服务集成（MCP）
- **GoogleDriveMCPToolkit**: Google Drive
- **GoogleGmailMCPToolkit**: Gmail
- **GoogleCalendarToolkit**: 日程管理
- **NotionMCPToolkit**: Notion 数据库

#### 协作工具类
- **HumanToolkit**: 人机交互确认
- **RAGToolkit**: 检索增强生成
- **SlackToolkit**: Slack 消息
- **LinkedInToolkit**: LinkedIn 操作

### 4.3 Toolkit 抽象接口

```python
# backend/app/utils/toolkit/abstract_toolkit.py
class AbstractToolkit:
    """Toolkit 抽象基类"""

    @classmethod
    def get_tools(cls) -> list[BaseTool]:
        """获取工具列表"""
        pass

    @classmethod
    def toolkit_name(cls) -> str:
        """返回工具包名称"""
        pass
```

### 4.4 Toolkit 使用示例

```python
# 创建 TerminalToolkit
terminal_toolkit = TerminalToolkit(
    working_directory="/path/to/project",
    persist_session=True
)

# 获取工具
tools = terminal_toolkit.get_tools()
# 返回: [
#   FunctionTool(name="execute_command"),
#   FunctionTool(name="list_files"),
#   ...
# ]

# 集成到 Agent
agent = ListenChatAgent(
    system_message=system_message,
    tools=tools,
    ...
)
```

---

## 5. Workforce 多智能体编排

### 5.1 Workforce 架构

```
                    ┌─────────────────────────┐
                    │       Workforce         │
                    │   (多智能体工作流引擎)    │
                    └────────────┬────────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
    ┌───────▼────────┐   ┌──────▼──────┐   ┌────────▼────────┐
    │  Coordinator   │   │Task Planner │   │Shared Task      │
    │   (项目经理)    │   │ (策略主管)  │   │   Channel       │
    └───────┬────────┘   └──────┬──────┘   └────────┬────────┘
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
┌───────▼──────┐      ┌─────────▼─────┐      ┌──────────▼─────┐
│  Developer   │      │    Browser     │      │   Document      │
│   Worker     │      │    Worker      │      │    Worker       │
└───────┬──────┘      └─────────┬─────┘      └──────────┬─────┘
        │                      │                       │
        └──────────────────────┼───────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Task Distribution  │
                    │  & Result Aggregation│
                    └─────────────────────┘
```

### 5.2 Shared Task Channel

**任务共享通道**是 Workforce 的核心通信机制：

```python
# backend/app/service/task.py
class SharedTaskChannel:
    """任务共享通道"""

    def __init__(self):
        self.task_queue = asyncio.Queue()
        self.result_store = {}

    async def put_task(self, task: Task):
        """发布任务到通道"""
        await self.task_queue.put(task)

    async def get_task(self, worker_id: str) -> Task:
        """Worker 获取任务"""
        return await self.task_queue.get()

    def put_result(self, task_id: str, result: Any):
        """存储任务结果"""
        self.result_store[task_id] = result
```

### 5.3 任务执行流程

```
用户提交任务
    │
    ▼
┌───────────────┐
│  Task Planner │ ← 分解复杂任务
│  (分解任务)    │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│  Coordinator  │ ← 根据能力分配给 Workers
│ (任务路由)     │
└───────┬───────┘
        │
    ┌───┴──────────────────────┐
    │                          │
┌───▼────────┐          ┌──────▼──────┐
│ Worker 1   │          │  Worker 2   │
│ (执行)     │          │  (执行)     │
└───┬────────┘          └──────┬──────┘
    │                          │
    └────────┬─────────────────┘
             │
        ┌────▼────────┐
        │  Shared     │
        │  Channel    │
        └────┬────────┘
             │
        ┌────▼────┐
        │  聚合   │
        │  结果   │
        └────┬────┘
             │
             ▼
        返回用户
```

### 5.4 失败处理机制

```python
class FailureHandler:
    """失败处理策略"""

    async def handle_failure(self, task: Task, error: Exception):
        retry_count = task.retry_count

        if retry_count < MAX_RETRY:
            # 策略 1: 分解重试
            if is_complex_task(task):
                subtasks = decompose_task(task)
                for subtask in subtasks:
                    await self.channel.put_task(subtask)

            # 策略 2: 升级处理
            else:
                specialized_worker = create_specialist_worker(task)
                await specialized_worker.execute(task)
        else:
            # 策略 3: 停止工作流
            await self.channel.stop_workflow(task.workflow_id)
```

---

## 6. 前端架构

### 6.1 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **React** | 18.3.1 | UI 框架 |
| **TypeScript** | 5.4.2 | 类型安全 |
| **Electron** | 33.2.0 | 桌面应用 |
| **Vite** | 5.4.11 | 构建工具 |
| **Zustand** | 5.0.4 | 状态管理 |
| **React Flow** | 12.6.4 | 工作流编辑器 |
| **Tailwind CSS** | 3.4.15 | 样式框架 |
| **Radix UI** | Latest | UI 组件库 |
| **Framer Motion** | 12.17.0 | 动画库 |
| **XTerm.js** | 5.5.0 | 终端模拟 |

### 6.2 组件架构

```
src/
├── components/
│   ├── ui/                    # 基础 UI 组件
│   │   ├── Button
│   │   ├── Dialog
│   │   ├── Dropdown
│   │   └── ...
│   ├── WorkFlow/              # 工作流编辑器
│   │   ├── Canvas.tsx         # React Flow 画布
│   │   ├── AgentNode.tsx      # Agent 节点
│   │   ├── Edge.tsx           # 连接线
│   │   └── Controls.tsx       # 控制面板
│   ├── ChatBox/               # 聊天界面
│   │   ├── ChatContainer.tsx
│   │   ├── MessageList.tsx
│   │   ├── InputArea.tsx
│   │   └── MessageItem/
│   │       ├── UserMessage.tsx
│   │       ├── AgentMessageCard.tsx
│   │       └── TaskStatusCard.tsx
│   ├── TerminalAgentWorkSpace/   # Developer 工作区
│   │   ├── TerminalPanel.tsx     # XTerm 终端
│   │   ├── FileTree.tsx          # 文件树
│   │   └── CodeEditor.tsx        # 代码编辑器
│   ├── BrowserAgentWorkSpace/    # Browser 工作区
│   │   ├── BrowserView.tsx       # 浏览器视图
│   │   └── SearchPanel.tsx       # 搜索面板
│   └── AddWorker/               # 添加 Worker
│       ├── WorkerConfig.tsx
│       └── ToolkitSelector.tsx
├── store/                      # Zustand 状态
│   ├── authStore.ts
│   ├── chatStore.ts
│   ├── projectStore.ts
│   └── globalStore.ts
├── pages/                      # 页面
│   ├── Home.tsx
│   ├── Workspace.tsx
│   └── Settings.tsx
├── hooks/                      # 自定义 Hooks
│   ├── useSSE.ts
│   ├── useAgent.ts
│   └── useTask.ts
├── api/                        # API 客户端
│   └── client.ts
├── lib/                        # 工具库
│   └── utils.ts
└── types/                      # 类型定义
    └── chatbox.d.ts
```

### 6.3 状态管理（Zustand）

```typescript
// src/store/chatStore.ts
interface ChatStore {
  // 聊天状态
  messages: Message[];
  agents: Agent[];
  tasks: TaskInfo[];

  // 操作
  addMessage: (message: Message) => void;
  updateAgent: (agentId: string, updates: Partial<Agent>) => void;

  // SSE 流
  connectStream: (taskId: string) => void;
  disconnectStream: () => void;
}

export const useChatStore = create<ChatStore>((set, get) => ({
  messages: [],
  agents: [],
  tasks: [],

  addMessage: (message) => set((state) => ({
    messages: [...state.messages, message]
  })),

  // ... 其他实现
}));
```

### 6.4 工作流编辑器（React Flow）

```typescript
// src/components/WorkFlow/Canvas.tsx
export function WorkflowCanvas() {
  const { nodes, edges, onNodesChange, onEdgesChange } = useWorkflowStore();

  return (
    <ReactFlow
      nodes={nodes}
      edges={edges}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      nodeTypes={agentNodeTypes}
    >
      <Controls />
      <MiniMap />
      <Background />
    </ReactFlow>
  );
}
```

### 6.5 SSE 实时通信

```typescript
// src/hooks/useSSE.ts
export function useSSE(taskId: string) {
  const [events, setEvents] = useState<TaskEvent[]>([]);

  useEffect(() => {
    const eventSource = new EventSource(
      `${API_BASE}/api/chat/stream?task_id=${taskId}`
    );

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);

      switch (data.action) {
        case 'activate_agent':
          handleAgentActivate(data);
          break;
        case 'activate_toolkit':
          handleToolkitActivate(data);
          break;
        case 'deactivate_agent':
          handleAgentDeactivate(data);
          break;
        // ... 其他事件
      }

      setEvents((prev) => [...prev, data]);
    };

    return () => eventSource.close();
  }, [taskId]);

  return events;
}
```

---

## 7. MCP（Model Context Protocol）集成

### 7.1 MCP 架构

```
┌─────────────────────────────────────────────────────┐
│                  Eigent Application                 │
└─────────────────────┬───────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌───▼──────┐ ┌───▼────────┐
│ MCP Server 1 │ │MCP Server│ │MCP Server 3│
│  (GitHub)    │ │2 (Gmail) │ │  (Notion)  │
└───────┬──────┘ └───┬──────┘ └───┬────────┘
        │            │            │
        └────────────┼────────────┘
                     │
            ┌────────▼────────┐
            │  MCP Agent      │
            │  (Factory)      │
            └─────────────────┘
```

### 7.2 MCP 配置

```json
// mcp_config.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "google-drive": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-google-drive"],
      "env": {
        "GOOGLE_CREDENTIALS": "${GOOGLE creds}"
      }
    }
  }
}
```

### 7.3 MCP Agent 工厂

```python
# backend/app/agent/factory/mcp.py
async def mcp_agent(options: Chat, mcp_config: dict) -> ListenChatAgent:
    """创建具有 MCP 工具的 Agent"""

    # 1. 加载 MCP 服务器
    mcp_toolkits = []
    for server_name, server_config in mcp_config.items():
        toolkit = await load_mcp_server(server_name, server_config)
        mcp_toolkits.append(toolkit)

    # 2. 获取工具
    tools = []
    for toolkit in mcp_toolkits:
        tools.extend(toolkit.get_tools())

    # 3. 创建 Agent
    return await agent_model(
        agent_name="mcp_agent",
        system_message=MCP_SYS_PROMPT,
        options=options,
        tools=tools,
        tool_names=[toolkit.toolkit_name() for toolkit in mcp_toolkits]
    )
```

---

## 8. 事件驱动架构

### 8.1 事件类型

```python
# backend/app/service/task.py
class Action(Enum):
    """任务事件类型"""
    activate_agent = "activate_agent"         # Agent 激活
    deactivate_agent = "deactivate_agent"     # Agent 停用
    activate_toolkit = "activate_toolkit"     # 工具激活
    deactivate_toolkit = "deactivate_toolkit" # 工具停用
    budget_not_enough = "budget_not_enough"   # 预算不足
    task_status_update = "task_status_update" # 任务状态更新
```

### 8.2 事件处理流程

```
Agent 执行
    │
    ▼
┌─────────────────┐
│  发送激活事件    │ → ActionActivateAgentData
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Task Channel   │
│  (事件队列)      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────────┐
│ SSE   │ │ 状态存储  │
│ 推送  │ │ (DB)      │
└───┬───┘ └──┬────────┘
    │        │
    ▼        ▼
前端更新   持久化
```

### 8.3 事件数据结构

```typescript
// 前端事件类型
interface AgentActivateEvent {
  action: 'activate_agent';
  data: {
    agent_name: string;
    process_task_id: string;
    agent_id: string;
    message: string;
  };
}

interface ToolkitActivateEvent {
  action: 'activate_toolkit';
  data: {
    agent_name: string;
    process_task_id: string;
    toolkit_name: string;
    method_name: string;
    message: string;  // JSON 格式的参数
  };
}

interface TaskStatusEvent {
  action: 'task_status_update';
  data: {
    task_id: string;
    status: 'pending' | 'running' | 'completed' | 'failed';
    progress: number;
  };
}
```

---

## 9. 数据模型

### 9.1 核心数据模型

```python
# backend/app/model/chat.py
class Chat(BaseModel):
    """聊天配置"""
    task_id: str
    user_id: str
    message: str
    project_id: str | None = None
    model_config: ModelConfig
    agent_tools: AgentToolsConfig | None = None

class ModelConfig(BaseModel):
    """模型配置"""
    platform: str  # "openai", "anthropic", "local", etc.
    model_type: str
    api_key: str | None = None
    base_url: str | None = None
    temperature: float = 0.7
    max_tokens: int = 4096

class AgentToolsConfig(BaseModel):
    """Agent 工具配置"""
    agent_type: AgentNameType
    tools: list[str]  # 工具列表
    mcp_tools: dict[str, Any] | None = None  # MCP 工具配置
```

### 9.2 前端类型定义

```typescript
// src/types/chatbox.d.ts
type AgentNameType =
  | 'developer_agent'
  | 'browser_agent'
  | 'document_agent'
  | 'multi_modal_agent'
  | 'social_media_agent';

interface Agent {
  agent_id: string;
  name: string;
  type: AgentNameType;
  status: AgentStatus;
  tasks: TaskInfo[];
  tools: string[];
  workerInfo?: {
    name: string;
    description: string;
    tools: any;
    mcp_tools: any;
    selectedTools: any;
  };
}

interface TaskInfo {
  task_id: string;
  status: TaskStatus;
  agent_name?: string;
  toolkit_name?: string;
  method_name?: string;
  message?: string;
  result?: any;
}

type AgentStatus = 'idle' | 'running' | 'completed' | 'failed';
type TaskStatus = 'pending' | 'running' | 'completed' | 'failed';
```

---

## 10. API 接口文档

### 10.1 聊天接口

```typescript
// 启动聊天任务
POST /api/chat/start
Request: {
  message: string;
  model_config: ModelConfig;
  agent_tools?: AgentToolsConfig;
}
Response: {
  task_id: string;
  status: string;
}

// SSE 流式响应
GET /api/chat/stream?task_id={task_id}
Response: Server-Sent Events stream

// 停止任务
POST /api/task/stop
Request: {
  task_id: string;
}
```

### 10.2 模型接口

```typescript
// 获取模型列表
GET /api/model/list
Response: {
  models: ModelInfo[];
}

// 获取模型配置
GET /api/model/config
Response: {
  platforms: PlatformInfo[];
  models: ModelConfig[];
}
```

### 10.3 工具接口

```typescript
// 获取可用工具列表
GET /api/tool/list
Response: {
  toolkits: ToolkitInfo[];
}

// 获取 MCP 服务器
GET /api/tool/mcp
Response: {
  mcp_servers: MCPServerInfo[];
}
```

---

## 11. 部署架构

### 11.1 本地部署

```
┌─────────────────────────────────────────┐
│           Desktop Application            │
│         (Electron + React)               │
└───────────────────┬─────────────────────┘
                    │ localhost
┌───────────────────▼─────────────────────┐
│         Local Backend (FastAPI)          │
│         ┌───────────────┐               │
│         │  CAMEL-AI     │               │
│         │  Workforce    │               │
│         └───────┬───────┘               │
│                 │                       │
│    ┌────────────┼────────────┐         │
│    │            │            │         │
┌───▼───┐   ┌────▼────┐   ┌───▼─────┐   │
│ vLLM  │   │ Ollama  │   │  Local  │   │
│       │   │         │   │ Models  │   │
└───────┘   └─────────┘   └─────────┘   │
└─────────────────────────────────────────┘
```

### 11.2 云连接模式

```
┌─────────────────────────────────────────┐
│      Desktop Application                │
└───────────────────┬─────────────────────┘
                    │ HTTPS
┌───────────────────▼─────────────────────┐
│       Eigent Cloud Backend              │
│         (User Data in Cloud)            │
└───────────────────┬─────────────────────┘
                    │
┌───────────────────▼─────────────────────┐
│         Cloud LLM APIs                  │
│    (OpenAI, Anthropic, etc.)            │
└─────────────────────────────────────────┘
```

### 11.3 Docker 部署

```yaml
# docker-compose.yml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://...
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./data:/app/data

  postgres:
    image: postgres:16
    environment:
      - POSTGRES_DB=eigent
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass

  redis:
    image: redis:alpine
```

---

## 12. 监控与遥测

### 12.1 OpenTelemetry 集成

```python
# backend/app/utils/telemetry/tracer.py
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

@tracer.start_as_current_span("agent_execution")
def execute_agent_task(agent, task):
    # Agent 执行追踪
    with tracer.start_as_current_span("tool_execution"):
        result = agent.execute(task)
    return result
```

### 12.2 指标收集

```python
# 收集指标
metrics = {
    "agent_execution_time": execution_duration,
    "tool_usage_count": tool_calls,
    "token_consumption": total_tokens,
    "task_success_rate": success_rate,
}
```

---

## 13. 安全与认证

### 13.1 认证流程

```
用户登录
    │
    ▼
┌───────────────┐
│ OAuth 2.0     │
│ (GitHub/Google)│
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ 获取 Access   │
│    Token      │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ 存储到        │
│ LocalStorage  │
└───────┬───────┘
        │
        ▼
后续请求携带 Token
```

### 13.2 API 密钥管理

```python
# 加密存储 API 密钥
from cryptography.fernet import Fernet

def encrypt_api_key(key: str) -> str:
    fernet = Fernet(ENCRYPTION_KEY)
    return fernet.encrypt(key.encode()).decode()

def decrypt_api_key(encrypted: str) -> str:
    fernet = Fernet(ENCRYPTION_KEY)
    return fernet.decrypt(encrypted.encode()).decode()
```

---

## 14. 性能优化

### 14.1 后端优化

- **异步 I/O**: 全面使用 `async/await`
- **连接池**: 数据库连接复用
- **缓存**: Redis 缓存频繁访问数据
- **流式响应**: SSE 减少延迟

### 14.2 前端优化

- **代码分割**: React.lazy + Suspense
- **虚拟化长列表**: react-window
- **防抖/节流**: 优化输入处理
- **Memo 优化**: React.memo 减少重渲染

---

## 15. 未来规划

### 15.1 短期目标

- [ ] 支持更多本地模型（Llama 3, Mistral）
- [ ] 增强可视化工作流编辑器
- [ ] 优化 SSE 连接稳定性
- [ ] 添加更多 MCP 集成

### 15.2 长期目标

- [ ] 插件系统
- [ ] Agent Marketplace
- [ ] 分布式 Worker 执行
- [ ] 移动端支持

---

## 附录

### A. 环境变量

```bash
# 后端
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk--ant-

# 前端
VITE_API_BASE_URL=http://localhost:8000
VITE_APP_TITLE=Eigent
```

### B. 依赖版本

详见 `package.json` 和 `pyproject.toml`。

### C. 参考资源

- [CAMEL-AI 文档](https://www.camel-ai.org/)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [FastAPI 文档](https://fastapi.tiangolo.com/)
- [React Flow 文档](https://reactflow.dev/)

---

**文档版本**: 1.0
**最后更新**: 2025-02-06
**维护者**: Eigent Team
