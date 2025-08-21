# Quick Start Guide

Get Echo up and running in under 5 minutes! This guide will walk you through installing Echo, running your first
multi-agent workflow, and understanding the basics.

## Prerequisites

- Git
- Conda (miniconda or Anaconda)
- Poetry
- Python 3.13 (we'll create a Conda env for this)
- OpenAI API key (or other LLM provider)

## Installation

### 1. Clone the Repository (with submodules)

```bash
git clone --recurse-submodules https://github.com/jonaskahn/echo.git
cd echo

# If you already cloned without submodules
git submodule update --init --recursive
```

### 2. Create Conda Environment

```bash
conda create -n echo python=3.13 -y
conda activate echo
```

### 3. Install Dependencies with Poetry

```bash
# If Poetry isn't installed in your environment yet
python -m pip install --upgrade pip
pip install poetry

# Install Echo with local path dependencies (SDK, Plugins)
poetry install --only main
( cd sdk && poetry install )
( cd plugins && poetry install )
```

### 4. Set Up Environment

Create a `.env` file in the root directory:

```bash
# Echo 🤖 Multi-agents AI Framework Environment Configuration
ECHO_APP_NAME="Echo 🤖 Multi-agents AI Framework"
ECHO_DEBUG=false

# LLM Provider Configuration
ECHO_DEFAULT_LLM_PROVIDER=openai

# API Keys
ECHO_OPENAI_API_KEY=your_actual_openai_api_key_here

# Plugin Configuration
ECHO_PLUGINS_DIR=./plugins/src/echo_plugins

# API Server Configuration
ECHO_API_HOST=0.0.0.0
ECHO_API_PORT=8000
```

## First Run

### 1. Start the System

```bash
python -m echo
```

You should see output like:

```text
INFO - Starting Echo 🤖 Multi-agents AI Framework...
INFO - Discovered 2 plugin directories across 1 plugin directories
INFO - Registered plugin: mathematics v0.1.0
INFO - Registered plugin: websearch v0.1.0
INFO - Successfully created plugin bundle: mathematics v0.1.0
INFO - Successfully created plugin bundle: websearch v0.1.0
INFO - Echo 🤖 Multi-agents AI Framework started successfully
```

### 2. Test the API

Open a new terminal and test the system:

```bash
# Check system status
curl http://localhost:8000/api/v1/status

# List available plugins
curl http://localhost:8000/api/v1/plugins

# Chat (single turn)
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Hello, Echo!"
  }'
```

## Understanding the System

### What Just Happened?

1. **Plugin Discovery**: Echo automatically discovered your math and search agent plugins
2. **Plugin Loading**: Each plugin was validated and loaded into the system
3. **Agent Creation**: Plugin agents were instantiated with their tools
4. **API Startup**: The REST API server started on port 8000

### Available Plugins

- **Math Agent**: Handles mathematical calculations and problem-solving
- **Search Agent**: Performs web searches and information retrieval

## Try It Out

### Math Agent Example

```bash
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Use math to compute 15 * 23."
  }'
```

### 3. Launch the Debug UI (optional)

Run the built-in Streamlit UI for quick local debugging:

```bash
# In a new terminal (with the same virtualenv activated)
# Optionally point the UI to a remote/backend URL
# export ECHO_API_BASE_URL=http://localhost:8000

streamlit run src/echo/ui/app.py
```

Then open your browser at:

```text
http://localhost:8501
```

Notes:

- The UI uses `ECHO_API_BASE_URL` (default `http://localhost:8000`).
- It keeps a local thread in session for multi-turn chats.

### Search Agent Example

```bash
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What are the latest developments in AI?"
  }'
```

## Next Steps

Now that you have Echo running, explore:

- **[Plugin Development](../plugins/overview.md)** - Learn to build custom agents
- **[Core Concepts](../concepts/architecture.md)** - Understand the system architecture

## Troubleshooting

### Common Issues

**Port already in use:**

```bash
# Change port in .env file
ECHO_API_PORT=8001
```

**Plugin not found:**

```bash
# Check plugin directory path in .env
ECHO_PLUGINS_DIR=./plugins/src/echo_plugins
```

**API key issues:**

```bash
# Ensure your API key is set correctly
echo $ECHO_OPENAI_API_KEY
```

### Getting Help

- Review [Notices & Tips](notices.md)

## Congratulations

You have successfully set up Echo. The framework is designed to be simple and powerful—feel free to experiment and build
your own plugins.
