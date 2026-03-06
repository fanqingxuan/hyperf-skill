---
name: eino
description: "Eino LLM/AI application development framework assistant (Golang). Use when the user needs to: (1) Build AI agents, (2) Create LLM applications, (3) Implement tool calling, (4) Build multi-agent systems, (5) Create workflows with Graph/Compose, (6) Implement streaming, (7) Human-in-the-loop patterns, or any other Eino framework development tasks. Triggers on phrases like \"Eino 开发\", \"创建 Agent\", \"LLM 应用\", \"AI Agent\", \"Eino 框架\", \"构建智能体\"."
---

# Eino 框架开发指南

Eino 是字节跳动开源的 Golang LLM/AI 应用开发框架，专注于 Agent 开发、工作流编排、工具调用。

## 环境要求

- **Go 版本**: 1.18+
- **代码规范**: golangci-lint

## 快速开始

### 安装

```bash
go get github.com/cloudwego/eino
go get github.com/cloudwego/eino-ext
```

### 项目初始化

```bash
# 创建项目
mkdir my-eino-app && cd my-eino-app
go mod init my-eino-app

# 安装依赖
go get github.com/cloudwego/eino
go get github.com/cloudwego/eino-ext/components/model/openai
```

## 核心示例

### 1. ChatModelAgent（基础 Agent）

最简单的对话 Agent：

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
)

func main() {
    ctx := context.Background()

    // 配置 ChatModel
    chatModel, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })
    if err != nil {
        panic(err)
    }

    // 创建 Agent
    agent, err := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })
    if err != nil {
        panic(err)
    }

    // 运行 Agent
    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: agent})
    iter := runner.Query(ctx, "Hello, who are you?")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }
        fmt.Println(event.Message.Content)
    }
}
```

### 2. 带工具的 Agent

添加工具调用能力：

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/components/tool"
)

// 定义天气工具
type WeatherTool struct{}

func (w *WeatherTool) Info(ctx context.Context) (*tool.Info, error) {
    return &tool.Info{
        Name: "get_weather",
        Desc: "Get current weather for a location",
        ParamsOneOf: tool.NewParamsOneOfByParams(
            map[string]*tool.ParameterInfo{
                "location": {
                    Type: "string",
                    Desc: "City name",
                    Required: true,
                },
            },
        ),
    }, nil
}

func (w *WeatherTool) InvokableRun(ctx context.Context, argumentsInJSON string) (string, error) {
    // 实际应该调用天气 API
    return fmt.Sprintf("Weather in %s: Sunny, 25°C", argumentsInJSON), nil
}

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    // 创建工具
    weatherTool := &WeatherTool{}

    // 创建带工具的 Agent
    agent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
        ToolsConfig: adk.ToolsConfig{
            ToolsNodeConfig: compose.ToolsNodeConfig{
                Tools: []tool.BaseTool{weatherTool},
            },
        },
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: agent})
    iter := runner.Query(ctx, "What's the weather in Beijing?")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }
        fmt.Println(event.Message.Content)
    }
}
```

### 3. DeepAgent（复杂任务）

处理复杂多步骤任务：

```go
package main

import (
    "context"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
    "github.com/cloudwego/eino/adk/deep"
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/components/tool"
)

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    // 创建子 Agent
    researchAgent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })

    codeAgent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })

    // 创建 DeepAgent
    deepAgent, _ := deep.New(ctx, &deep.Config{
        ChatModel: chatModel,
        SubAgents: []adk.Agent{researchAgent, codeAgent},
        ToolsConfig: adk.ToolsConfig{
            ToolsNodeConfig: compose.ToolsNodeConfig{
                Tools: []tool.BaseTool{
                    // shellTool, pythonTool, webSearchTool
                },
            },
        },
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: deepAgent})
    iter := runner.Query(ctx, "Analyze the sales data and generate a report")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }
        // 处理事件
    }
}
```

### 4. Graph 编排

精确控制执行流程：

```go
package main

import (
    "context"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/compose"
)

type Input struct {
    Text string
}

type Output struct {
    Result string
}

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    // 创建 Graph
    graph := compose.NewGraph[*Input, *Output]()

    // 添加节点
    graph.AddLambdaNode("validate", func(ctx context.Context, input *Input) (*Input, error) {
        // 验证逻辑
        return input, nil
    })

    graph.AddChatModelNode("generate", chatModel)

    graph.AddLambdaNode("format", func(ctx context.Context, input *Input) (*Output, error) {
        // 格式化逻辑
        return &Output{Result: input.Text}, nil
    })

    // 添加边
    graph.AddEdge(compose.START, "validate")
    graph.AddEdge("validate", "generate")
    graph.AddEdge("generate", "format")
    graph.AddEdge("format", compose.END)

    // 编译并运行
    runnable, _ := graph.Compile(ctx)
    result, _ := runnable.Invoke(ctx, &Input{Text: "Hello"})

    println(result.Result)
}
```

### 5. Graph 转工具

将工作流暴露为 Agent 工具：

```go
package main

import (
    "context"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/components/tool"
    "github.com/cloudwego/eino/components/tool/graphtool"
)

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    // 创建 Graph
    graph := compose.NewGraph[*Input, *Output]()
    // ... 添加节点和边

    // 将 Graph 转换为工具
    graphTool, _ := graphtool.NewInvokableGraphTool(
        graph,
        "data_pipeline",
        "Process and validate data",
    )

    // Agent 使用该工具
    agent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
        ToolsConfig: adk.ToolsConfig{
            ToolsNodeConfig: compose.ToolsNodeConfig{
                Tools: []tool.BaseTool{graphTool},
            },
        },
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: agent})
    iter := runner.Query(ctx, "Process the data using the pipeline")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }
        // 处理事件
    }
}
```

### 6. 流式处理

处理流式输出：

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
)

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    agent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: agent})

    // 流式查询
    iter := runner.Query(ctx, "Write a long story")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }

        // 实时输出流式内容
        if event.Message != nil && event.Message.Content != "" {
            fmt.Print(event.Message.Content)
        }
    }
}
```

### 7. 回调处理

添加日志、追踪、指标：

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
    "github.com/cloudwego/eino/callbacks"
)

// 自定义回调处理器
type LoggingHandler struct {
    callbacks.Handler
}

func (h *LoggingHandler) OnStart(ctx context.Context, info *callbacks.RunInfo, input callbacks.CallbackInput) context.Context {
    fmt.Printf("Start: %s\n", info.Name)
    return ctx
}

func (h *LoggingHandler) OnEnd(ctx context.Context, info *callbacks.RunInfo, output callbacks.CallbackOutput) context.Context {
    fmt.Printf("End: %s\n", info.Name)
    return ctx
}

func (h *LoggingHandler) OnError(ctx context.Context, info *callbacks.RunInfo, err error) context.Context {
    fmt.Printf("Error: %s - %v\n", info.Name, err)
    return ctx
}

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    // 添加回调处理器
    handler := &LoggingHandler{}
    ctx = callbacks.CtxWithHandlers(ctx, handler)

    agent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{Agent: agent})
    iter := runner.Query(ctx, "Hello")

    for {
        event, ok := iter.Next()
        if !ok {
            break
        }
        fmt.Println(event.Message.Content)
    }
}
```

### 8. 人机协作（Interrupt/Resume）

实现人工审批流程：

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/adk"
)

func main() {
    ctx := context.Background()

    chatModel, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        Model:  "gpt-4o",
        APIKey: os.Getenv("OPENAI_API_KEY"),
    })

    agent, _ := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Model: chatModel,
    })

    runner := adk.NewRunner(ctx, adk.RunnerConfig{
        Agent: agent,
        // 配置检查点存储
    })

    // 第一次运行
    iter := runner.Query(ctx, "Create a report")

    var checkpointID string
    for {
        event, ok := iter.Next()
        if !ok {
            break
        }

        // 检测到需要人工确认
        if event.NeedHumanInput {
            checkpointID = event.CheckpointID
            fmt.Println("Waiting for human approval...")
            break
        }
    }

    // 人工确认后恢复
    if checkpointID != "" {
        // 获取人工输入
        humanInput := "Approved"

        // 从检查点恢复
        iter = runner.Resume(ctx, checkpointID, humanInput)

        for {
            event, ok := iter.Next()
            if !ok {
                break
            }
            fmt.Println(event.Message.Content)
        }
    }
}
```

## 核心概念

### 1. Component（组件）

可复用的构建块：

- **ChatModel**: 大语言模型接口
- **Tool**: 工具/函数调用
- **Retriever**: 检索器
- **Embedding**: 向量嵌入
- **ChatTemplate**: 提示词模板

### 2. Agent（智能体）

具有自主决策能力的实体：

- **ChatModelAgent**: 基础 Agent，处理简单对话和工具调用
- **DeepAgent**: 复杂任务 Agent，支持任务分解和多 Agent 协作
- **自定义 Agent**: 实现 `adk.Agent` 接口

### 3. Graph（图）

节点和边组成的执行流程：

```go
graph := compose.NewGraph[InputType, OutputType]()
graph.AddLambdaNode("node1", func1)
graph.AddChatModelNode("node2", chatModel)
graph.AddEdge("node1", "node2")
```

### 4. Tool（工具）

Agent 可以调用的功能：

```go
type MyTool struct{}

func (t *MyTool) Info(ctx context.Context) (*tool.Info, error) {
    return &tool.Info{
        Name: "my_tool",
        Desc: "Tool description",
        ParamsOneOf: tool.NewParamsOneOfByParams(params),
    }, nil
}

func (t *MyTool) InvokableRun(ctx context.Context, args string) (string, error) {
    // 工具逻辑
    return result, nil
}
```

### 5. Stream（流）

流式数据处理：

- 自动处理流式传输
- 连接、装箱、合并、复制流
- 组件只需实现适合的流式范式

### 6. Callback（回调）

切面编程机制：

```go
type MyHandler struct {
    callbacks.Handler
}

func (h *MyHandler) OnStart(ctx context.Context, info *callbacks.RunInfo, input callbacks.CallbackInput) context.Context {
    // 开始时的逻辑
    return ctx
}
```

### 7. Checkpoint（检查点）

状态持久化：

- 保存执行状态
- 支持中断恢复
- 人机协作

## 支持的模型

### ChatModel 实现

- **OpenAI**: GPT-4, GPT-3.5
- **Claude**: Claude 3 系列
- **Gemini**: Google Gemini
- **Ark**: 字节跳动火山引擎
- **Ollama**: 本地模型
- **自定义**: 实现 `ChatModel` 接口

### Embedding 实现

- **OpenAI**: text-embedding-ada-002
- **Ark**: 火山引擎 Embedding
- **自定义**: 实现 `Embedding` 接口

### Retriever 实现

- **Elasticsearch**: 全文检索
- **OpenSearch**: 向量检索
- **自定义**: 实现 `Retriever` 接口

## 开发最佳实践

1. **组件复用**: 使用 eino-ext 提供的官方组件
2. **错误处理**: 始终检查错误返回值
3. **上下文传递**: 正确使用 context.Context
4. **流式优先**: 优先使用流式 API 提升用户体验
5. **回调监控**: 使用回调机制进行日志和监控
6. **工具设计**: 工具应该单一职责、参数清晰
7. **Graph 编排**: 复杂流程使用 Graph 而非硬编码
8. **检查点**: 长时间任务使用检查点支持恢复
9. **类型安全**: 充分利用 Go 的类型系统
10. **测试**: 为 Agent 和工具编写单元测试

## 常用命令

```bash
# 代码检查
golangci-lint run ./...

# 格式化代码
gofmt -s -w .

# 整理导入
goimports -w .

# 运行测试
go test ./...

# 构建
go build -o app main.go

# 运行
./app
```

## 官方资源

- **GitHub**: https://github.com/cloudwego/eino
- **文档**: https://www.cloudwego.io/docs/eino/
- **示例**: https://github.com/cloudwego/eino-examples
- **扩展**: https://github.com/cloudwego/eino-ext
- **社区**: 飞书群（见 README）

## 使用场景

1. **对话机器人**: 客服、问答、知识库查询
2. **工具调用**: 天气查询、计算器、数据库操作
3. **数据分析**: 数据处理、报表生成、可视化
4. **代码生成**: 代码助手、自动化脚本
5. **多 Agent 协作**: 任务分解、专业分工
6. **工作流自动化**: 业务流程、ETL 任务
7. **人机协作**: 审批流程、交互式任务
8. **RAG 应用**: 知识库检索增强生成

## 注意事项

1. **API Key 安全**: 不要硬编码 API Key，使用环境变量
2. **并发控制**: 注意 Go 的并发特性，避免竞态条件
3. **资源释放**: 及时释放资源，避免内存泄漏
4. **错误处理**: 不要忽略错误，做好错误处理和日志
5. **性能优化**: 使用流式 API、批处理、缓存等优化性能
6. **版本兼容**: 注意 Eino 和 eino-ext 的版本兼容性
7. **模型选择**: 根据任务复杂度选择合适的模型
8. **成本控制**: 监控 API 调用次数和 Token 使用量

## 官方示例索引

详细示例请参考 `references/docs_examples/` 目录：

### 📦 ADK (Agent Development Kit)

| 目录 | 名称 | 说明 |
|------|------|------|
| adk/helloworld | Hello World Agent | 最简单的 Agent 示例，展示如何创建基础对话 Agent |
| adk/intro/chatmodel | ChatModel Agent | ChatModelAgent 使用和 Interrupt 机制 |
| adk/intro/custom | 自定义 Agent | 实现符合 ADK 定义的自定义 Agent |
| adk/intro/workflow | Workflow Agents | Loop、Parallel、Sequential Agent 模式 |
| adk/intro/session | Session 管理 | 通过 Session 在多个 Agent 间传递数据和状态 |
| adk/intro/transfer | Agent 转移 | ChatModelAgent 的 Transfer 能力，Agent 间任务转移 |
| adk/intro/http-sse-service | HTTP SSE 服务 | 将 ADK Runner 暴露为 HTTP 服务（Server-Sent Events） |
| adk/human-in-the-loop | 人机协作 | 8 个示例：审批、审核编辑、反馈循环、追问、Supervisor |
| adk/multiagent | 多 Agent 协作 | Supervisor、Plan-Execute-Replan、Deep Agents、Excel Agent |
| adk/common/tool/graphtool | GraphTool | 将 Graph/Chain/Workflow 封装为 Agent 工具 |

### 🔗 Compose (编排)

| 目录 | 名称 | 说明 |
|------|------|------|
| compose/chain | Chain | 使用 compose.Chain 进行顺序编排（Prompt + ChatModel） |
| compose/graph | Graph | 图编排：状态图、工具调用 Agent、异步节点、中断机制 |
| compose/workflow | Workflow | 工作流：字段映射、纯数据流、纯控制流、静态值、流式处理 |
| compose/batch | BatchNode | 批量处理组件，支持并发控制和中断恢复 |

### 🌊 Flow (流程模块)

| 目录 | 名称 | 说明 |
|------|------|------|
| flow/agent/react | ReAct Agent | ReAct Agent，包含记忆、动态选项、未知工具处理 |
| flow/agent/multiagent | Multi-Agent | Host Multi-Agent（日记助手）、Plan-Execute 模式 |
| flow/agent/manus | Manus Agent | 基于 Eino 实现的 Manus Agent（参考 OpenManus） |
| flow/agent/deer-go | Deer-Go | 参考 deer-flow 的 Go 实现，支持研究团队协作 |

### 🧩 Components (组件)

| 目录 | 名称 | 说明 |
|------|------|------|
| components/model | Model | A/B 测试路由、cURL 风格的 HTTP 传输日志 |
| components/retriever | Retriever | 多查询检索、路由检索 |
| components/tool | Tool | JSON Schema 工具、MCP 工具、中间件（错误移除、JSON 修复） |
| components/document | Document | 自定义解析器、扩展解析器、文本解析器 |
| components/prompt | Prompt | Chat Prompt 模板示例 |
| components/lambda | Lambda | Lambda 函数组件示例 |

### 🚀 QuickStart (快速开始)

| 目录 | 名称 | 说明 |
|------|------|------|
| quickstart/chat | Chat 快速开始 | 最基础的 LLM 对话示例（模板、生成、流式输出） |
| quickstart/eino_assistant | Eino 助手 | 完整的 RAG 应用（知识索引、Agent 服务、Web 界面） |
| quickstart/todoagent | Todo Agent | 简单的 Todo 管理 Agent 示例 |

### 🛠️ DevOps (开发运维)

| 目录 | 名称 | 说明 |
|------|------|------|
| devops/debug | 调试工具 | Eino 调试功能，支持 Chain 和 Graph 调试 |
| devops/visualize | 可视化工具 | 将 Graph/Chain/Workflow 渲染为 Mermaid 图表 |

## 参考文档

详细文档请参考 `references/` 目录：

- **框架概览**: [references/overview.md](references/overview.md) - Eino 框架完整梳理，包含核心概念、使用场景、学习路径
- **README**: [references/readme.md](references/readme.md) - 官方 README 文档（英文）
- **官方示例**: [references/docs_examples/](references/docs_examples/) - 官方示例代码库
- **示例说明**: [references/docs_examples/COOKBOOK.md](references/docs_examples/COOKBOOK.md) - 每个示例的详细说明
- **官方文档**: https://www.cloudwego.io/docs/eino/ - 在线文档
- **扩展组件**: https://github.com/cloudwego/eino-ext - 组件实现库
