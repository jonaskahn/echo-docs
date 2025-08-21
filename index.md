# Echo 🤖 Multi-agents AI Framework

<div class="grid cards" markdown>

- **[Quick Start](getting-started/quick-start.md)**

  Get up and running with Echo in minutes

- **[Core Concepts](concepts/architecture.md)**

  Understand the architecture and philosophy

- **[Plugin Development](plugins/overview.md)**

  Build and deploy custom AI agents

</div>

## What is Echo?

**Echo** is a powerful, open-source multi-agent orchestration system designed to simplify the development and deployment
of AI agent workflows. Built with modern Python technologies like LangChain and LangGraph, Echo provides a robust
foundation for creating intelligent, collaborative AI systems.

## Philosophy

Echo is built on three core principles:

### 1. **Simplicity First**

- Clean, intuitive APIs that get out of your way
- Minimal boilerplate code for maximum productivity
- Clear separation of concerns between core system and plugins

### 2. **Plugin-Driven Architecture**

- Everything is a plugin - from agents to tools
- Hot-reloadable plugins for rapid development
- True decoupling between core system and custom functionality

### 3. **Production Ready**

- Built for scale with enterprise-grade reliability
- Comprehensive monitoring and observability
- Docker support and cloud-native deployment

## Architecture Overview

```mermaid
flowchart TD
    U[Client Request] --> API[FastAPI API]
    API --> COORD["Coordinator (LangGraph)"]
    COORD --> PM[SDK Plugin Manager]

    PM -->|discovers| B1[Plugin bundle: math_agent]
    PM -->|discovers| B2[Plugin bundle: search_agent]

    subgraph B1
      A1[[AgentNode]]
      T1[[ToolNode]]
      A1 -- "should_continue = continue" --> T1
      T1 --> A1
    end

    subgraph B2
      A2[[AgentNode]]
      T2[[ToolNode]]
      A2 -- "should_continue = continue" --> T2
      T2 --> A2
    end

    %% Agent returns control to coordinator when not calling tools
    A1 -- "should_continue = back" --> COORD
    A2 -- "should_continue = back" --> COORD

    %% LLM binding used inside AgentNode execution
    A1 --> LLM[(LLM Model)]
    A2 --> LLM

    COORD --> API
    API --> U
```

## Key Features

- **Multi-Agent Orchestration**: Coordinate multiple AI agents in complex workflows
- **Plugin System**: Extend functionality without touching core code
- **Hot Reloading**: Update plugins without restarting the system
- **LLM Agnostic**: Support for OpenAI, Anthropic, Google, and more
- **REST API**: Full HTTP API for integration with any system
- **Operational API**: REST endpoints for chat, plugins, status, health
- **Comprehensive Logging**: Built-in observability and debugging tools

## Quick Example

```python
from echo_sdk import BasePlugin, PluginMetadata, BasePluginAgent


class MyPlugin(BasePlugin):
    @staticmethod
    def get_metadata() -> PluginMetadata:
        return PluginMetadata(
            name="my_agent",
            version="0.1.0",
            description="My custom AI agent",
            capabilities=["custom_task"]
        )

    @staticmethod
    def create_agent() -> BasePluginAgent:
        return MyAgent(MyPlugin.get_metadata())


class MyAgent(BasePluginAgent):
    def get_tools(self):
        return [my_custom_tool]

    def get_system_prompt(self):
        return "You are a helpful AI assistant."
```

## What's Next?

- **[Quick Start Guide](getting-started/quick-start.md)** - Get Echo running in 5 minutes
- **[Plugin Development](plugins/overview.md)** - Learn to build custom agents
- **[Architecture](concepts/architecture.md)** - Understand the system design
- **[Environment](deployment/environment.md)** - Configure your setup

## Contributing

Echo is open source and welcomes contributions! Check out our [contributing guide](contributing/development.md) to get
started.

## License

Echo is licensed under the MIT License - see the [LICENSE](https://github.com/jonaskahn/echo/blob/main/LICENSE) file for
details.
