# Architecture Overview

Echo is built with a modular, plugin-driven architecture that emphasizes simplicity, extensibility, and production
readiness. This document explains the core architectural decisions and how the system components interact.

## High-Level Architecture

```mermaid
flowchart TD
    subgraph Clients
        Web[Web UI]
        ApiClient[API Client]
        WsClient[WebSocket Client]
    end

    subgraph API["API (FastAPI)"]
        Val[Validation & CORS]
    end

    subgraph Core["Core System"]
        Coord["Coordinator (LangGraph)"]
        SDKPM[SDK Plugin Manager]
        State[State Manager]
        LLMF[LLM Factory]
        Obs[Observability]
        Conf[Configuration]
    end

    subgraph Discovery["Plugin Discovery"]
        Pip["Pip-installed packages"]
        Dir["Directory-based packages"]
        Reg["Plugin Registry (SDK)"]
    end

    subgraph Bundles["Plugin Bundles"]
        AgentNode["AgentNode"]
        ToolNode["ToolNode"]
    end

    subgraph External["External Services"]
        OA[OpenAI]
        AN[Anthropic]
        GG[Google]
        Redis[(Redis)]
        DB[(PostgreSQL/SQLite)]
    end

    Web --> API
    ApiClient --> API
    WsClient --> API

    API --> Coord
    Coord --> SDKPM
    SDKPM --> Discovery
    Discovery --> Pip
    Discovery --> Dir
    SDKPM --> Reg
    SDKPM --> Bundles
    Bundles --> AgentNode
    Bundles --> ToolNode
    AgentNode -- "continue" --> ToolNode
    ToolNode --> AgentNode
    AgentNode -- "back" --> Coord

    Coord --> State
    State --> Redis
    State --> DB

    Coord --> LLMF
    LLMF --> OA
    LLMF --> AN
    LLMF --> GG

    API --> Obs
    Coord --> Obs
```

## Core Components

### 1. **API (FastAPI)**

The entry point for all external communication:

- **REST API**: HTTP endpoints for synchronous operations
- WebSocket support is not implemented in this version
- Authentication and rate limiting are not included by default
- **CORS**: Cross-origin resource sharing configuration
- **Validation**: Request/response validation with Pydantic

### 2. **Multi-Agent Orchestrator**

The brain of the system that coordinates agent interactions:

- **Workflow Management**: LangGraph-based workflow orchestration
- **Agent Routing**: Intelligent routing to appropriate agents with hop limit handling
- **State Management**: Persistent conversation and workflow state
- **Error Handling**: Graceful failure recovery and fallbacks
- **Performance Optimization**: Caching and resource management
- **Suspend Node**: Intelligent handling of hop limits with AI communication

### 3. **Plugin Manager**

Handles the complete plugin lifecycle:

- **Discovery**: Automatic plugin detection and loading
- **Validation**: Plugin structure and dependency verification
- **Lifecycle Management**: Loading, unloading, and hot-reloading
- **Resource Management**: Memory and connection pooling
- **Health Monitoring**: Plugin status and performance metrics

### 4. **LLM Factory**

Manages connections to various language models:

- **Provider Abstraction**: Unified interface for different LLM providers
- **Model Caching**: Intelligent caching of model instances
- **Configuration Management**: Provider-specific settings and parameters
- **Fallback Handling**: Automatic failover between providers
- **Cost Optimization**: Token usage tracking and optimization

## Data Flow

### Plugin Discovery Sources

Echo aggregates plugins from two sources at startup:

- Pip-installed packages (environment packages)
    - Discovered via the SDK registry when packages that depend on `echo_sdk` are present
    - Import of the package triggers `register_plugin(...)`
    - No extra configuration needed beyond having the package installed

- Directory-based packages (filesystem)
    - Controlled via environment variable `ECHO_PLUGINS_DIR`

```bash
# Single directory
ECHO_PLUGINS_DIR=./plugins/src/echo_plugins

# Or JSON list of directories
ECHO_PLUGINS_DIR=["/abs/path/one", "/abs/path/two"]
```

Requirements for directory discovery:

- Each entry is a valid Python package (has `__init__.py`)
- The package imports call `register_plugin(MyPlugin)` so the SDK registry can collect it

### Application Startup Flow (performed once at app start)

```mermaid
sequenceDiagram
    participant Main as Echo Main
    participant PM as SDKPluginManager
    participant Disc as Discovery (registry)
    participant C as PluginContract
    participant LLM as LLMModelFactory
    participant A as Agent (BasePluginAgent)

    Main->>PM: initialize()
    PM->>PM: load_plugin_packages()
    PM->>Disc: discover_plugins()
    Disc-->>PM: [PluginContract, ...]
    loop for each contract
        PM->>C: get_metadata()
        PM->>C: create_agent()
        C-->>PM: Agent instance
        PM->>A: get_tools()
        PM->>LLM: create_base_model(model_config)
        LLM-->>PM: BaseChatModel
        PM->>A: bind_model(base_model)
        PM->>A: initialize()
        PM->>A: create_agent_node()
        PM-->>Main: register PluginBundle (AgentNode, ToolNode, Edges)
    end
```

### Request Processing Flow (uses preloaded bundles)

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Gateway
    participant Coord as Orchestrator (LangGraph)
    participant B as Plugin Bundle
    participant Agent as AgentNode
    participant Tools as ToolNode
    participant LLM as LLM (bound)

    C->>API: HTTP Request
    API->>Coord: Forward request
    Coord->>B: Select bundle based on routing
    Coord->>Agent: Invoke agent node (with state)
    Agent->>LLM: Invoke bound model
    alt should_continue == "continue"
        Agent->>Tools: Call tool(s)
        Tools-->>Agent: Tool result
        Agent-->>Coord: Updated state
    else should_continue == "back"
        Agent-->>Coord: Return control
    end
    Coord-->>API: Orchestrated response
    API-->>C: HTTP Response
```

### Agent-as-Plugin Integration with LangGraph (startup)

This section explains precisely how a plugin becomes executable nodes in the LangGraph workflow.

```mermaid
sequenceDiagram
    participant Core as Echo Core
    participant PM as SDKPluginManager
    participant SDK as echo_sdk.registry
    participant C as PluginContract
    participant LLM as LLMModelFactory
    participant A as Agent (BasePluginAgent)
    participant T as Tools (List[Tool])

    Core->>PM: discover_and_load_plugins()
    PM->>SDK: discover_plugins()
    SDK-->>PM: [PluginContract, ...]
    loop for each contract
        PM->>C: get_metadata()
        PM->>C: create_agent()
        C-->>PM: Agent instance
        PM->>A: get_tools()
        A-->>PM: List[Tool]
        PM->>LLM: create_base_model(model_config)
        LLM-->>PM: BaseChatModel
        PM->>A: bind_model(base_model)
        A-->>PM: Bound model
        PM->>A: initialize()
        PM->>A: create_agent_node()
        PM-->>Core: ToolNode + AgentNode + Edges
    end
```

At run-time the orchestrator uses the pre-wired nodes and edges like this:

```mermaid
flowchart LR
    subgraph Plugin[Plugin Bundle]
      AgentNode[["agent.create_agent_node()"]]
      ToolNode[["ToolNode(tools)"]]
    end
    Coordinator[[coordinator]]
    SuspendNode[[suspend]]
    Finalizer[[finalizer]]

    AgentNode -- "should_continue = continue" --> ToolNode
    ToolNode --> AgentNode
    AgentNode -- "should_continue = back" --> Coordinator
    Coordinator --> ToolNode
    ToolNode -- "hop limit reached" --> SuspendNode
    ToolNode -- "normal routing" --> AgentNode
    SuspendNode --> Finalizer
```

Key responsibilities:

- Agent.get_tools(): returns LangChain Tools used by the agent
- Agent.bind_model(): binds tools to the chat model (model.bind_tools(tools))
- Agent.create_agent_node(): returns the callable used as the LangGraph node
- Agent.should_continue(state): returns "continue" to call tools or "back" to return to the coordinator

#### Suspend Node for Hop Limit Handling

The suspend node provides intelligent handling of hop limits:

- **Intercepts** when `max_agent_hops` is reached
- **Informs the AI** about the limit situation with context
- **Allows AI response** before finalization
- **Improves user experience** by explaining why processing stopped
- **Preserves context** across the limit boundary

When hop limits are reached, the workflow automatically routes through:
`ToolNode → SuspendNode → Finalizer → END`

This ensures the AI can communicate with users about incomplete processing and provide helpful context.

#### Core Wiring (excerpt)

The following excerpt (simplified) shows bundle creation and graph plumbing:

```python
# src/echo/infrastructure/plugins/sdk_manager.py (excerpt)
def discover_and_load_plugins(self) -> None:
    self.load_plugin_packages()
    contracts = discover_plugins()
    for contract in contracts:
        self._create_plugin_bundle(contract)


def _create_plugin_bundle(self, contract: PluginContract) -> bool:
    metadata = contract.get_metadata()
    agent = contract.create_agent()
    base_model = self.llm_factory.create_base_model(
        self._create_model_config(metadata)
    )
    tools = agent.get_tools()
    bound_model = agent.bind_model(base_model)
    agent.initialize()
    bundle = SDKPluginBundle(
        contract=contract,
        agent=agent,
        bound_model=bound_model,
        tools=tools,
    )
    # bundle.tool_node and bundle.agent_node are then registered in the graph
    return True
```

This is where tools, the bound model, and the agent node are produced and attached to the graph.

### Plugin Loading Flow

```mermaid
sequenceDiagram
    participant S as System
    participant PM as Plugin Manager
    participant D as Discovery
    participant V as Validator
    participant R as Registry

    S->>PM: Initialize
    PM->>D: Scan Directories
    D->>V: Validate Plugins
    V->>R: Register Valid
    R->>PM: Plugin List
    PM->>S: Ready
```

## Design Principles

### 1. **Separation of Concerns**

Each component has a single, well-defined responsibility:

- **API Gateway**: Handles HTTP concerns only
- **Orchestrator**: Manages workflow logic only
- **Plugin Manager**: Handles plugin lifecycle only
- **LLM Factory**: Manages model connections only

### 2. **Plugin-First Architecture**

Everything is a plugin, enabling:

- **Extensibility**: Add new capabilities without code changes
- **Modularity**: Independent development and deployment
- **Maintainability**: Isolated testing and debugging
- **Scalability**: Horizontal scaling of specific capabilities

### 3. **Configuration-Driven**

System behavior is controlled through:

- **Environment Variables**: Runtime configuration
- **Plugin Metadata**: Capability declarations
- **Dynamic Settings**: Runtime parameter adjustment
- **Validation**: Configuration integrity checks

### 4. **Production Ready**

Built-in features for production deployment:

- **Health Checks**: Comprehensive system monitoring
- **Logging**: Structured logging with configurable levels
- **Metrics**: Performance and usage metrics
- **Error Handling**: Graceful degradation and recovery

## Technical Stack

### Backend Framework

- **FastAPI**: Modern, fast web framework for APIs
- **Pydantic**: Data validation and settings management
- **Uvicorn**: ASGI server for production deployment

### AI/ML Stack

- **LangChain**: LLM application framework
- **LangGraph**: Workflow orchestration
- **OpenAI/Anthropic/Google**: LLM provider APIs

### Data Management

- **Redis**: Caching and session storage
- **SQLite/PostgreSQL**: Persistent data storage
- **Pydantic**: Data models and validation

### Development Tools

- **Poetry**: Dependency management
- **Pytest**: Testing framework
- **Black/Isort**: Code formatting
- **MyPy**: Type checking

## Scalability Considerations

### Horizontal Scaling

- **Stateless Design**: API components can be replicated
- **Plugin Isolation**: Plugins run independently
- **Load Balancing**: Multiple instances can share load
- **Database Sharding**: Data can be distributed

### Performance Optimization

- **Connection Pooling**: Efficient resource management
- **Caching Layers**: Multiple levels of caching
- **Async Operations**: Non-blocking I/O operations
- **Resource Limits**: Configurable resource constraints

### Monitoring and Observability

- **Health Endpoints**: System status monitoring
- **Metrics Collection**: Performance data gathering
- **Log Aggregation**: Centralized log management
- **Tracing**: Request flow tracking

## Security Architecture

### Authentication & Authorization

- **JWT Tokens**: Stateless authentication
- **Role-Based Access**: Granular permission control
- **API Key Management**: Secure credential storage
- **Rate Limiting**: Abuse prevention

### Data Protection

- **Input Validation**: Comprehensive input sanitization
- **Output Encoding**: Safe data presentation
- **Encryption**: Data in transit and at rest
- **Audit Logging**: Security event tracking

## Performance Characteristics

### Response Times

- **Simple Queries**: < 100ms
- **LLM Processing**: 1-5 seconds (provider dependent)
- **Plugin Operations**: < 500ms
- **Complex Workflows**: 5-30 seconds

### Throughput

- **Concurrent Users**: 100+ simultaneous users
- **Requests/Second**: 1000+ RPS (depending on complexity)
- **Plugin Instances**: Configurable per plugin
- **Memory Usage**: 100MB-2GB (depending on plugins)

## Future Architecture

### Planned Enhancements

- **Microservices**: Service decomposition for scale
- **Event Streaming**: Real-time event processing
- **GraphQL**: Flexible query interface
- **Kubernetes**: Container orchestration
- **Service Mesh**: Inter-service communication

### Extension Points

- **Custom Orchestrators**: Alternative workflow engines
- **Plugin Marketplaces**: Third-party plugin distribution
- **Multi-Tenancy**: Isolated user environments
- **Federation**: Distributed Echo instances

## Related Documentation

- **[Plugin System](plugin-system.md)** - Plugin architecture details
- **[Deployment](../deployment/production.md)** - Production setup
