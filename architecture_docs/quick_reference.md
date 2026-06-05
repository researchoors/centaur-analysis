# Centaur Quick Reference

## System At-A-Glance

**Centaur** is an AI-first platform for orchestrating intelligent agent executions across a rich ecosystem of 72+ specialized tools. The platform enables building, deploying, and managing AI agents through Slack, APIs, and web interfaces with enterprise-grade security and scalability.

## Core Components

### 🏗️ **Platform Services** (3 components)
- **`centaur-api`** - Central orchestration hub, workflow engine, sandbox management
- **`slackbot`** - Slack integration bridge with real-time streaming  
- **`centaur-sdk`** - Foundational library for tool development

### 🛠️ **Tool Ecosystem** (72 components)
- **Business** (3): Ashby, Attio, Pylon
- **Communication** (2): Telegram, Twitter
- **Crypto/DeFi** (20): Alchemy, CoinGecko, DefiLlama, Polymarket, etc.
- **Infrastructure** (11): Grafana, Sentry, CloudWatch, PostHog, etc.
- **Media** (4): Nano-banana, Veo3, Transcriber
- **Productivity** (10): Linear, Notion, Google Workspace, Slack, etc.
- **Research** (22): Websearch, Crunchbase, Congress.gov, YouTube, etc.

### 📚 **Documentation**
- **`centaur-docs`** - Developer documentation with React + Vocs

## Key URLs & Endpoints

### API Endpoints
```
# Agent Operations
POST   /agent/execute        # Execute agents with streaming
POST   /agent/spawn          # Create new sessions
POST   /agent/message        # Send messages to sessions

# Workflow Management  
POST   /workflows/runs       # Create/start workflows
GET    /workflows/runs       # List workflow runs

# Tool Execution
POST   /tools/{tool_name}    # Execute specific tools

# File Operations
POST   /attachments          # Upload files
```

### Slack Integration
```
# Event Processing
POST   /api/slack/events     # Process Slack events
POST   /api/slack/messages   # Post/update messages

# Streaming
POST   /api/slack/streams/start   # Start message streams
POST   /api/slack/streams/append  # Append to streams

# Agent Sessions
POST   /api/slack/agent-sessions       # Create sessions
POST   /api/slack/agent-sessions/{id}/text  # Stream text
POST   /api/slack/agent-sessions/{id}/done  # Complete sessions
```

## Tech Stack

### Core Technologies
| Component | Language | Framework | Database | 
|-----------|----------|-----------|----------|
| API | Python | FastAPI | PostgreSQL + pgVector |
| Slackbot | TypeScript | Hono | - |
| Tools | Python | Typer/Rich | Various APIs |
| Docs | TypeScript | React + Vocs | - |

### Infrastructure
- **Container Orchestration**: Kubernetes
- **Security**: Iron-proxy for secret management
- **Observability**: OpenTelemetry + Grafana
- **AI**: Anthropic Claude, OpenAI GPT

## Architecture Patterns

### 🔄 **Execution Flow**
```
Slack Event → Slackbot → Centaur API → K8s Sandbox → Tool Execution → Results
```

### 🔐 **Security Model**
```
Agent Request → Tool Server → Iron Proxy → External API
                     ↓             ↓
              Secret Stubs → Secret Injection
```

### 📊 **Data Flow**
```
Request → FastAPI → PostgreSQL → Kubernetes → Container → Tools → Response
```

## Key Configuration

### Environment Variables
```bash
# Core API
DATABASE_URL="postgresql://..."
CENTAUR_API_KEY="..."

# Slack Integration  
SLACK_BOT_TOKEN="xoxb-..."
SLACK_SIGNING_SECRET="..."

# AI Services
ANTHROPIC_API_KEY="sk-..."
OPENAI_API_KEY="sk-..."

# Observability
OTEL_EXPORTER_OTLP_ENDPOINT="..."
```

### Secret Management
```python
# Tool context (server mode - returns placeholder)
from centaur_sdk import secret
api_key = secret("SOME_API_KEY")

# CLI mode (reads from environment)
export SOME_API_KEY="actual_value"
```

## Tool Development

### Basic Tool Structure
```python
import typer
from rich import print

def main(query: str):
    """Tool description for AI agents."""
    # Tool logic here
    print(f"Processing: {query}")

if __name__ == "__main__":
    typer.run(main)
```

### Integration Patterns
```python
from centaur_sdk import secret, save_attachment

# Secure API access
api_key = secret("API_KEY", default="fallback")

# File handling
attachment = save_attachment("data.json", json_data)
```

## Common Operations

### Agent Execution
```bash
# Via API
curl -X POST /agent/execute \
  -H "Authorization: Bearer $API_KEY" \
  -d '{"message": "Search for crypto trends"}'

# Via Slack  
@centaur search for crypto trends
```

### Tool Usage
```bash
# Direct tool execution
curl -X POST /tools/websearch \
  -d '{"query": "AI trends 2024"}'

# In workflow
{
  "tool": "websearch",
  "args": {"query": "AI trends 2024"}
}
```

### File Operations
```bash
# Upload attachment
curl -X POST /attachments \
  -F "file=@document.pdf" \
  -F "name=analysis_doc"
```

## Development Workflow

### 1. Tool Development
```bash
# Create tool
mkdir tools/category/my-tool
cd tools/category/my-tool

# Setup
poetry init
poetry add typer rich httpx

# Develop
# Edit pyproject.toml, implement tool logic
```

### 2. Testing
```bash
# Unit tests
pytest tests/

# Integration testing
python -m my_tool --help
```

### 3. Deployment
```bash
# Auto-discovery via tool manager
# Hot-reload in development
# Container deployment in production
```

## Troubleshooting

### Common Issues

**Agent not responding**
- Check sandbox pod logs: `kubectl logs -l app=agent-sandbox`
- Verify tool server connectivity
- Check PostgreSQL connection

**Tool errors** 
- Verify API credentials in iron-proxy
- Check tool logs in container
- Validate input parameters

**Slack integration issues**
- Verify webhook URL configuration  
- Check signature verification
- Review event deduplication

### Debug Commands
```bash
# Check API health
curl /health

# View agent logs
kubectl logs -f deployment/centaur-api

# Database queries
psql $DATABASE_URL -c "SELECT * FROM agent_sessions LIMIT 5;"

# Tool discovery
curl /tools
```

## Performance Guidelines

### Optimization Tips
- Use streaming for long-running operations
- Implement proper error handling and retries
- Cache frequently accessed data
- Monitor resource usage in containers

### Scaling Considerations  
- API and Slackbot services are stateless
- Database connection pooling is critical
- Kubernetes horizontal pod autoscaling
- Monitor OpenTelemetry metrics

## Security Best Practices

### Tool Development
- Never hardcode secrets
- Use centaur-sdk secret management
- Validate all inputs
- Follow principle of least privilege

### Infrastructure
- Container security policies
- Network isolation via policies
- Regular secret rotation
- Audit logging enabled

---

## Quick Start Commands

```bash
# Deploy local development
docker-compose up -d

# Run tool directly  
cd tools/research/websearch
python -m websearch search "query"

# Test API endpoint
curl -X POST localhost:8000/agent/execute \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, world!"}'

# Check system health
curl localhost:8000/health
```

For detailed information, see the complete [Architecture Documentation](./architecture.md).