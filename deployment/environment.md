# Environment Configuration

Echo uses environment variables for configuration, making it easy to deploy across different environments. This guide
covers all available configuration options and best practices aligned with the current codebase.

## Configuration Methods

Echo supports multiple configuration methods:

1. **Environment Variables** (recommended for production)
2. **`.env` file** (convenient for development)
3. **Command line arguments** (for testing and debugging)

## Configuration Variables

### Application Settings

| Variable        | Default                          | Description                                |
|-----------------|----------------------------------|--------------------------------------------|
| `ECHO_APP_NAME` | "Echo Multi-agents AI Framework" | Application name displayed in logs and API |
| `ECHO_DEBUG`    | `false`                          | Enable debug mode                          |

### LLM Provider Configuration

| Variable                          | Default                    | Description                                                                                      |
|-----------------------------------|----------------------------|--------------------------------------------------------------------------------------------------|
| `ECHO_DEFAULT_LLM_PROVIDER`       | `openai`                   | Default LLM provider (`openai`, `azure-openai`/`azure`, `anthropic`/`claude`, `google`/`gemini`) |
| `ECHO_OPENAI_DEFAULT_MODEL`       | `gpt-4.1`                  | Default model when provider is `openai` or `azure`/`azure-openai`                                |
| `ECHO_ANTHROPIC_DEFAULT_MODEL`    | `claude-sonnet-4-20250514` | Default model when provider is `anthropic`/`claude`                                              |
| `ECHO_GEMINI_DEFAULT_MODEL`       | `gemini-2.5-flash`         | Default model when provider is `google`/`gemini`                                                 |
| `ECHO_DEFAULT_LLM_TEMPERATURE`    | `0.1`                      | Default sampling temperature for LLMs                                                            |
| `ECHO_DEFAULT_LLM_CONTEXT_WINDOW` | `32000`                    | Default context window size                                                                      |

> Notes
>
> - `ECHO_DEFAULT_LLM_MODEL` is not used.
> - For Azure OpenAI, default model resolution uses `ECHO_OPENAI_DEFAULT_MODEL`. You can still set
    `ECHO_AZURE_OPENAI_DEFAULT_MODEL`, but it is not used by the core default-model resolver.

### API Keys

| Variable                    | Required | Description                         |
|-----------------------------|----------|-------------------------------------|
| `ECHO_OPENAI_API_KEY`       | Yes\*    | OpenAI API key for GPT models       |
| `ECHO_AZURE_OPENAI_API_KEY` | Yes\*    | Azure OpenAI API key                |
| `ECHO_ANTHROPIC_API_KEY`    | Yes\*    | Anthropic API key for Claude models |
| `ECHO_GOOGLE_API_KEY`       | Yes\*    | Google API key for Gemini models    |

\*At least one API key is required based on your chosen provider.

### Azure OpenAI Settings

When using `azure-openai` (alias: `azure`) as the provider, configure the following:

| Variable                        | Default              | Description                          |
|---------------------------------|----------------------|--------------------------------------|
| `ECHO_AZURE_OPENAI_ENDPOINT`    | —                    | Azure OpenAI endpoint URL            |
| `ECHO_AZURE_OPENAI_API_VERSION` | `2024-02-15-preview` | Azure OpenAI API version             |
| `ECHO_AZURE_OPENAI_DEPLOYMENT`  | —                    | Azure OpenAI deployment (model name) |

### Plugin Configuration

| Variable           | Default                          | Description                                            |
|--------------------|----------------------------------|--------------------------------------------------------|
| `ECHO_PLUGINS_DIR` | `["./plugins/src/echo_plugins"]` | Plugin directory path(s). Accepts a path or JSON list. |

Examples:

```bash
# Single directory
ECHO_PLUGINS_DIR=./plugins/src/echo_plugins

# JSON list of directories
ECHO_PLUGINS_DIR=["/abs/path/one", "/abs/path/two"]
```

### API Server Configuration

| Variable            | Default   | Description             |
|---------------------|-----------|-------------------------|
| `ECHO_API_HOST`     | `0.0.0.0` | API server host address |
| `ECHO_API_PORT`     | `8000`    | API server port number  |
| `ECHO_CORS_ORIGINS` | `["*"]`   | CORS allowed origins    |

### Session Configuration

| Variable                   | Default | Description                    |
|----------------------------|---------|--------------------------------|
| `ECHO_SESSION_TIMEOUT`     | `3600`  | Session timeout in seconds     |
| `ECHO_MAX_SESSION_HISTORY` | `100`   | Maximum session history length |

### Processing Configuration

| Variable                     | Default | Description                                                                  |
|------------------------------|---------|------------------------------------------------------------------------------|
| `ECHO_MAX_AGENT_HOPS`        | `25`    | Maximum agent switches before suspend/finalization                           |
| `ECHO_MAX_TOOL_HOPS`         | `50`    | Maximum tool calls per request before suspend/finalization                   |
| `ECHO_GRAPH_RECURSION_LIMIT` | `50`    | Maximum LangGraph steps per request (prevents graph recursion/infinite loop) |

### Conversation Storage Configuration

#### Backend Selection

| Variable                            | Default  | Description                                            |
|-------------------------------------|----------|--------------------------------------------------------|
| `ECHO_CONVERSATION_STORAGE_BACKEND` | `memory` | Conversation backend (`memory`, `redis`, `postgresql`) |

#### PostgreSQL Configuration

| Variable                     | Default | Description                                |
|------------------------------|---------|--------------------------------------------|
| `ECHO_POSTGRES_URL`          | —       | PostgreSQL connection URL (asyncpg format) |
| `ECHO_POSTGRES_POOL_SIZE`    | `20`    | PostgreSQL connection pool size            |
| `ECHO_POSTGRES_MAX_OVERFLOW` | `30`    | PostgreSQL connection pool max overflow    |

#### Redis Configuration

| Variable               | Default | Description                |
|------------------------|---------|----------------------------|
| `ECHO_REDIS_URL`       | `None`  | Redis connection URL       |
| `ECHO_REDIS_POOL_SIZE` | `20`    | Redis connection pool size |

### Persistence Configuration (Chat Memory/Checkpoints)

| Variable                                            | Default      | Description                                                                           |
|-----------------------------------------------------|--------------|---------------------------------------------------------------------------------------|
| `ECHO_PERSISTENCE_TYPE`                             | `checkpoint` | Persistence mode: `checkpoint` or `memory`                                            |
| `ECHO_PERSISTENCE_CHECKPOINT_LAYER`                 | `redis`      | Backend for checkpoints: `redis`, `postgres`, or `sqlite` (falls back to in-memory)   |
| `ECHO_PERSISTENCE_CHECKPOINT_REDIS_TTL_MINUTES`     | `1440`       | Default TTL for Redis checkpoints (minutes)                                           |
| `ECHO_PERSISTENCE_CHECKPOINT_REDIS_REFRESH_ON_READ` | `true`       | Whether to refresh TTL on read for Redis checkpoints                                  |
| `ECHO_PERSISTENCE_MEMORY_LAYER`                     | `redis`      | Backend for active memory: `redis`, `postgres`, or `sqlite` (falls back to in-memory) |

Aliases supported: `anthropic` ≡ `claude`, `google` ≡ `gemini`, `azure-openai` ≡ `azure`.

## Storage and Persistence Options

### Conversation Storage Options

#### 1. Memory (Development)

```bash
ECHO_CONVERSATION_STORAGE_BACKEND=memory
# Fast development and testing
# No external dependencies
# Data lost on restart
```

#### 2. Redis (High-performance, ephemeral)

```bash
ECHO_CONVERSATION_STORAGE_BACKEND=redis
ECHO_REDIS_URL=redis://localhost:6379/0
# Sub-millisecond reads/writes
# Persistence depends on Redis configuration
```

#### 3. PostgreSQL (Production)

```bash
ECHO_CONVERSATION_STORAGE_BACKEND=postgresql
ECHO_POSTGRES_URL=postgresql+asyncpg://user:pass@host:port/database
ECHO_POSTGRES_POOL_SIZE=20
ECHO_POSTGRES_MAX_OVERFLOW=30
# ACID transactions, complex queries, backup/recovery
```

### Persistence Options (Chat Memory/Checkpoints)

#### 1. In-Memory (Zero dependencies)

```bash
ECHO_PERSISTENCE_TYPE=memory
# No external services
# Memory lost on restart
```

#### 2. Redis (Recommended for production checkpoints)

```bash
ECHO_PERSISTENCE_TYPE=checkpoint
ECHO_PERSISTENCE_CHECKPOINT_LAYER=redis
ECHO_PERSISTENCE_MEMORY_LAYER=redis
ECHO_REDIS_URL=redis://localhost:6379/0
# Distributed, TTL support, high performance
```

#### 3. PostgreSQL/SQLite (Enterprise/Portable)

```bash
# PostgreSQL
ECHO_PERSISTENCE_TYPE=checkpoint
ECHO_PERSISTENCE_CHECKPOINT_LAYER=postgres
ECHO_PERSISTENCE_MEMORY_LAYER=postgres
ECHO_POSTGRES_URL=postgresql+asyncpg://user:pass@host:5432/echo

# SQLite (portable, local testing)
ECHO_PERSISTENCE_TYPE=checkpoint
ECHO_PERSISTENCE_CHECKPOINT_LAYER=sqlite
ECHO_PERSISTENCE_MEMORY_LAYER=sqlite
```

## Environment File Setup

### Development Environment (`.env`)

```bash
# Echo Multi-agents AI Framework Environment Configuration
ECHO_APP_NAME="Echo Multi-agents AI Framework (Dev)"
ECHO_DEBUG=true

# LLM Provider Configuration
ECHO_DEFAULT_LLM_PROVIDER=openai
# Optionally pin the default model for the chosen provider
# ECHO_OPENAI_DEFAULT_MODEL=gpt-4.1
# ECHO_ANTHROPIC_DEFAULT_MODEL=claude-sonnet-4-20250514
# ECHO_GEMINI_DEFAULT_MODEL=gemini-2.5-flash

# API Keys
ECHO_OPENAI_API_KEY=sk-your-openai-key-here

# Conversation Storage (Development - No external dependencies)
ECHO_CONVERSATION_STORAGE_BACKEND=memory

# Persistence (keep everything in-memory for zero-deps dev)
ECHO_PERSISTENCE_TYPE=memory

# Plugin Configuration
ECHO_PLUGINS_DIR=./plugins/src/echo_plugins

# API Server Configuration
ECHO_API_HOST=127.0.0.1
ECHO_API_PORT=8000

# Session Configuration
ECHO_SESSION_TIMEOUT=1800
ECHO_MAX_SESSION_HISTORY=50

# Processing Configuration
ECHO_MAX_AGENT_HOPS=25
ECHO_MAX_TOOL_HOPS=50
ECHO_GRAPH_RECURSION_LIMIT=50

# --- Azure OpenAI (alternative) ---
# ECHO_DEFAULT_LLM_PROVIDER=azure-openai
# ECHO_AZURE_OPENAI_API_KEY=your-azure-openai-key
# ECHO_AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
# ECHO_AZURE_OPENAI_API_VERSION=2024-02-15-preview
# ECHO_AZURE_OPENAI_DEPLOYMENT=gpt-4o
```

### Production Environment

```bash
# Production Configuration
ECHO_APP_NAME="Echo Multi-agents AI Framework"
ECHO_DEBUG=false

# LLM Provider Configuration
ECHO_DEFAULT_LLM_PROVIDER=openai
# Provider-specific default model (optional)
# ECHO_OPENAI_DEFAULT_MODEL=${OPENAI_MODEL:-gpt-4.1}

# API Keys (use secure secret management)
ECHO_OPENAI_API_KEY=${OPENAI_API_KEY}

# Conversation Storage (Production)
ECHO_CONVERSATION_STORAGE_BACKEND=postgresql
ECHO_POSTGRES_URL=postgresql+asyncpg://echo:password@db.internal:5432/echo_prod
ECHO_POSTGRES_POOL_SIZE=20
ECHO_POSTGRES_MAX_OVERFLOW=30

# Persistence (Recommended: Redis)
ECHO_PERSISTENCE_TYPE=checkpoint
ECHO_PERSISTENCE_CHECKPOINT_LAYER=redis
ECHO_PERSISTENCE_MEMORY_LAYER=redis
ECHO_REDIS_URL=redis://redis:6379/0

# Plugin Configuration
ECHO_PLUGINS_DIR=/opt/echo/plugins

# API Server Configuration
ECHO_API_HOST=0.0.0.0
ECHO_API_PORT=8000

# Session Configuration
ECHO_SESSION_TIMEOUT=3600
ECHO_MAX_SESSION_HISTORY=100

# Processing Configuration
ECHO_MAX_AGENT_HOPS=25
ECHO_MAX_TOOL_HOPS=50
ECHO_GRAPH_RECURSION_LIMIT=50

# --- Azure OpenAI (alternative) ---
# ECHO_DEFAULT_LLM_PROVIDER=azure-openai
# ECHO_AZURE_OPENAI_API_KEY=${AZURE_OPENAI_API_KEY}
# ECHO_AZURE_OPENAI_ENDPOINT=${AZURE_OPENAI_ENDPOINT}
# ECHO_AZURE_OPENAI_API_VERSION=2024-02-15-preview
# ECHO_AZURE_OPENAI_DEPLOYMENT=${AZURE_OPENAI_DEPLOYMENT}
```

## Docker Environment

### Docker Compose Example

```yaml
version: "3.8"

services:
  echo:
    build: .
    ports:
      - "8000:8000"
    environment:
      - ECHO_APP_NAME=Echo Multi-agents AI Framework
      - ECHO_DEBUG=false
      - ECHO_DEFAULT_LLM_PROVIDER=openai
      # Optionally pin default model for provider
      # - ECHO_OPENAI_DEFAULT_MODEL=gpt-4.1
      - ECHO_OPENAI_API_KEY=${OPENAI_API_KEY}
      - ECHO_PLUGINS_DIR=/opt/echo/plugins
      - ECHO_API_HOST=0.0.0.0
      - ECHO_API_PORT=8000
      # Conversation storage
      - ECHO_CONVERSATION_STORAGE_BACKEND=postgresql
      - ECHO_POSTGRES_URL=postgresql+asyncpg://echo:password@postgres:5432/echo
      # Persistence (Redis)
      - ECHO_PERSISTENCE_TYPE=checkpoint
      - ECHO_PERSISTENCE_CHECKPOINT_LAYER=redis
      - ECHO_PERSISTENCE_MEMORY_LAYER=redis
      - ECHO_REDIS_URL=redis://redis:6379/0
      - ECHO_MAX_AGENT_HOPS=25
      - ECHO_MAX_TOOL_HOPS=50
      - ECHO_GRAPH_RECURSION_LIMIT=50
      # Azure OpenAI (alternative)
      # - ECHO_DEFAULT_LLM_PROVIDER=azure-openai
      # - ECHO_AZURE_OPENAI_API_KEY=${AZURE_OPENAI_API_KEY}
      # - ECHO_AZURE_OPENAI_ENDPOINT=${AZURE_OPENAI_ENDPOINT}
      # - ECHO_AZURE_OPENAI_API_VERSION=2024-02-15-preview
      # - ECHO_AZURE_OPENAI_DEPLOYMENT=${AZURE_OPENAI_DEPLOYMENT}
    volumes:
      - ./plugins:/opt/echo/plugins
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=echo
      - POSTGRES_USER=echo
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

## Database Setup Instructions

### PostgreSQL Setup

```bash
# 1. Install PostgreSQL
brew install postgresql  # macOS
sudo apt install postgresql  # Ubuntu

# 2. Create database and user
sudo -u postgres psql
CREATE DATABASE echo_db;
CREATE USER echo_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE echo_db TO echo_user;
\q

# 3. Set environment variable
export ECHO_POSTGRES_URL="postgresql+asyncpg://echo_user:secure_password@localhost:5432/echo_db"

# 4. Run migrations
poetry run alembic upgrade head
```

### Redis Setup

```bash
# 1. Install Redis
brew install redis  # macOS
sudo apt install redis-server  # Ubuntu

# 2. Start Redis
redis-server

# 3. Set environment variable
export ECHO_REDIS_URL="redis://localhost:6379/0"
```

## Performance Characteristics

### Conversation Storage Performance

| Backend        | Write Speed | Read Speed | Scale       | Consistency | Best For    |
|----------------|-------------|------------|-------------|-------------|-------------|
| **Memory**     | Fastest     | Fastest    | Single Node | Strong      | Development |
| **Redis**      | Fastest     | Fastest    | Horizontal  | Eventual    | Caching     |
| **PostgreSQL** | Fast        | Fast       | Vertical    | ACID        | Production  |

### Persistence Layer Performance

| Backend        | Latency | Distributed | TTL | Persistence | Best For       |
|----------------|---------|-------------|-----|-------------|----------------|
| **Memory**     | <1ms    | No          | N/A | No          | Local dev      |
| **Redis**      | ~1ms    | Yes         | Yes | Optional    | Production     |
| **PostgreSQL** | ~5ms    | Yes         | Yes | Yes         | Enterprise     |
| **SQLite**     | ~1-3ms  | No          | No  | Yes         | Portable/local |

### Dockerfile Environment

```dockerfile
FROM python:3.13-slim

# Set environment variables
ENV ECHO_APP_NAME="Echo Multi-agents AI Framework"
ENV ECHO_DEBUG=false
ENV ECHO_DEFAULT_LLM_PROVIDER=openai
ENV ECHO_API_HOST=0.0.0.0
ENV ECHO_API_PORT=8000

# ... rest of Dockerfile
```

## Cloud Deployment

### Kubernetes ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: echo-config
data:
  ECHO_APP_NAME: "Echo Multi-agents AI Framework"
  ECHO_DEBUG: "false"
  ECHO_DEFAULT_LLM_PROVIDER: "openai"
  # ECHO_OPENAI_DEFAULT_MODEL: "gpt-4.1"
  ECHO_PLUGINS_DIR: "/opt/echo/plugins"
  ECHO_API_HOST: "0.0.0.0"
  ECHO_API_PORT: "8000"
  ECHO_SESSION_TIMEOUT: "3600"
  ECHO_MAX_AGENT_HOPS: "25"
  ECHO_MAX_TOOL_HOPS: "50"
  ECHO_GRAPH_RECURSION_LIMIT: "50"
  # Conversation Storage
  ECHO_CONVERSATION_STORAGE_BACKEND: "postgresql"
  ECHO_POSTGRES_URL: "postgresql+asyncpg://echo:password@postgres:5432/echo"
  # Persistence
  ECHO_PERSISTENCE_TYPE: "checkpoint"
  ECHO_PERSISTENCE_CHECKPOINT_LAYER: "redis"
  ECHO_PERSISTENCE_MEMORY_LAYER: "redis"
  ECHO_REDIS_URL: "redis://redis:6379/0"
# --- Azure OpenAI (alternative) ---
#  ECHO_DEFAULT_LLM_PROVIDER: "azure-openai"
#  ECHO_AZURE_OPENAI_ENDPOINT: "https://your-resource.openai.azure.com/"
#  ECHO_AZURE_OPENAI_API_VERSION: "2024-02-15-preview"
#  ECHO_AZURE_OPENAI_DEPLOYMENT: "gpt-4o"
---
apiVersion: v1
kind: Secret
metadata:
  name: echo-secrets
type: Opaque
data:
  ECHO_OPENAI_API_KEY: <base64-encoded-key>
  # ECHO_AZURE_OPENAI_API_KEY: <base64-encoded-key>
```

### AWS ECS Task Definition

```json
{
  "family": "echo",
  "containerDefinitions": [
    {
      "name": "echo",
      "image": "echo:latest",
      "environment": [
        {
          "name": "ECHO_APP_NAME",
          "value": "Echo Multi-agents AI Framework"
        },
        {
          "name": "ECHO_DEBUG",
          "value": "false"
        },
        {
          "name": "ECHO_DEFAULT_LLM_PROVIDER",
          "value": "openai"
        },
        {
          "name": "ECHO_CONVERSATION_STORAGE_BACKEND",
          "value": "postgresql"
        },
        {
          "name": "ECHO_PERSISTENCE_TYPE",
          "value": "checkpoint"
        },
        {
          "name": "ECHO_PERSISTENCE_CHECKPOINT_LAYER",
          "value": "redis"
        }
      ],
      "secrets": [
        {
          "name": "ECHO_OPENAI_API_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:openai-api-key"
        }
        /* Azure OpenAI (alternative)
        ,{ "name": "ECHO_DEFAULT_LLM_PROVIDER", "value": "azure-openai" },
        { "name": "ECHO_AZURE_OPENAI_API_KEY", "valueFrom": "arn:aws:secretsmanager:region:account:secret:azure-openai-api-key" },
        { "name": "ECHO_AZURE_OPENAI_ENDPOINT", "value": "https://your-resource.openai.azure.com/" },
        { "name": "ECHO_AZURE_OPENAI_API_VERSION", "value": "2024-02-15-preview" },
        { "name": "ECHO_AZURE_OPENAI_DEPLOYMENT", "value": "gpt-4o" }
        */
      ]
    }
  ]
}
```

## Security Best Practices

### API Key Management

1. **Never commit API keys to version control**
2. **Use environment variables or secret management systems**
3. **Rotate keys regularly**
4. **Use least-privilege access**

### Environment Separation

```bash
# Development
.env.development

# Staging
.env.staging

# Production
.env.production
```

### Secret Management

```bash
# Use external secret managers
export ECHO_OPENAI_API_KEY=$(aws secretsmanager get-secret-value --secret-id openai-api-key --query SecretString --output text)

# Or use Docker secrets
echo $OPENAI_API_KEY | docker secret create openai-api-key -
```

## Configuration Validation

### Check Current Configuration

```bash
poetry run python -c "
from src.echo.config.settings import Settings
settings = Settings()
print(f'App Name: {settings.app_name}')
print(f'Debug: {settings.debug}')
print(f'LLM Provider: {settings.default_llm_provider}')
print(f'Conversation Storage: {settings.conversation_storage_backend}')
print(f'Persistence Type: {settings.persistence_type}')
print(f'Checkpoint Layer: {settings.persistence_checkpoint_layer}')
print(f'Memory Layer: {settings.persistence_memory_layer}')
print(f'PostgreSQL: {\"configured\" if settings.postgres_url else \"not configured\"}')
print(f'Redis: {\"configured\" if settings.redis_url else \"not configured\"}')
"
```

### Test Database Connections

```bash
# Test PostgreSQL connection
poetry run python -c "
import asyncio
from src.echo.infrastructure.database.connection import initialize_databases
from src.echo.config.settings import Settings

async def test():
    settings = Settings(postgres_url='postgresql+asyncpg://user:pass@localhost/echo')
    cm = await initialize_databases(settings)
    health = await cm.health_check()
    print(f'PostgreSQL: {health[\"postgres\"][\"status\"]}')

asyncio.run(test())
"

# Test Redis connection (persistence/Redis backend)
poetry run python -c "
import asyncio
import redis.asyncio as redis

async def test():
    client = redis.from_url('redis://localhost:6379/0')
    await client.ping()
    print('Redis: healthy')
    await client.aclose()

asyncio.run(test())
"
```

### Validate Configuration on Startup

Echo validates configuration on startup:

```python
from echo.config.settings import Settings

# This will validate all configuration
settings = Settings()

# Check if required settings are present
if not settings.validate_provider_credentials(settings.default_llm_provider):
    raise ValueError(f"Missing credentials for {settings.default_llm_provider}")
```

## Troubleshooting Configuration

### Common Issues

**Plugin directory not found:**

```bash
# Check if directory exists
ls -la $ECHO_PLUGINS_DIR

# Verify path is correct
echo $ECHO_PLUGINS_DIR
```

**API key not working:**

```bash
# Test API key
curl -H "Authorization: Bearer $ECHO_OPENAI_API_KEY" \
  https://api.openai.com/v1/models
```

**Database connection issues:**

```bash
# Test PostgreSQL connection
psql $ECHO_POSTGRES_URL -c "SELECT 1;"

# Test Redis connection
redis-cli -u $ECHO_REDIS_URL ping
```

**Port already in use:**

```bash
# Check what's using the port
lsof -i :8000

# Change port in configuration
export ECHO_API_PORT=8001
```

### Debug Mode

Enable debug mode to see detailed configuration:

```bash
export ECHO_DEBUG=true
python -m echo.main
```

## Deployment Scenarios

### Development (No external dependencies)

```bash
# .env file
ECHO_CONVERSATION_STORAGE_BACKEND=memory
ECHO_PERSISTENCE_TYPE=memory
ECHO_OPENAI_API_KEY=your_key_here

# Start immediately
python -m echo start all
```

### Production (PostgreSQL + Redis)

```bash
# Setup steps:
# 1. Create PostgreSQL database
# 2. Run migrations: poetry run alembic upgrade head
# 3. Start Redis server
# 4. Start Echo: python -m echo start all
```

### Hybrid Testing (Mix backends)

```bash
# PostgreSQL for conversations, in-memory persistence
ECHO_CONVERSATION_STORAGE_BACKEND=postgresql
ECHO_POSTGRES_URL=postgresql+asyncpg://echo:password@localhost:5432/echo_test
ECHO_PERSISTENCE_TYPE=memory

# Memory conversations, Redis persistence
ECHO_CONVERSATION_STORAGE_BACKEND=memory
ECHO_PERSISTENCE_TYPE=checkpoint
ECHO_PERSISTENCE_CHECKPOINT_LAYER=redis
ECHO_PERSISTENCE_MEMORY_LAYER=redis
ECHO_REDIS_URL=redis://localhost:6379/1
```

## Configuration Behavior

The system automatically:

1. Reads configuration from environment variables
2. Selects appropriate providers/backends based on your settings
3. Handles connection pooling and health monitoring
4. Falls back gracefully to in-memory if databases are unavailable

## Next Steps

- **[Production Setup](production.md)** - Production deployment guide
