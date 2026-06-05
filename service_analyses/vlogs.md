# VLogs Component Analysis

## Overview

The vlogs component is a Python CLI tool that provides a command-line interface for interacting with VictoriaLogs, a log analytics platform. The component implements a comprehensive LogsQL query client with specialized methods for analyzing application logs, tool usage, and system performance metrics in a distributed microservices environment.

## Architecture

The component follows a clean separation of concerns with a **client-server architecture pattern**:

- **Client Layer** (`VictoriaLogsClient`): HTTP client for VictoriaLogs API interactions
- **CLI Layer** (`cli.py`): Command-line interface using Typer framework
- **Test Layer** (`test_client.py`, `tests/`): Unit tests and mocking infrastructure

The architecture emphasizes convenience methods for common log analysis patterns, particularly around thread tracing, tool analytics, and execution monitoring.

## Key Components

### VictoriaLogsClient (client.py)
The core client class providing HTTP API access to VictoriaLogs with comprehensive query capabilities. It includes basic LogsQL operations (query, field_names, field_values, streams) and specialized convenience methods for application monitoring.

### CLI Interface (cli.py)  
Typer-based command-line interface exposing four primary commands: `query` for LogsQL queries, `fields` for listing field names, `field-values` for field value enumeration, and `streams` for log stream discovery. All commands support JSON output and time range filtering.

### Query Builder Functions
Utility functions for constructing LogsQL queries, including `_quote_logsql_value()` for escaping special characters in field values and `_field_expr()` for building field expressions.

### Convenience Analytics Methods
Specialized methods for common monitoring tasks: `thread_logs()` and `thread_trace()` for distributed tracing, `tool_analytics()` for tool usage analysis, `execution_summaries()` for workflow monitoring, and `service_health()` for error rate tracking.

### Time Handling Utilities
Helper methods for time range processing, including `_time_prefix()` for LogsQL time expressions, `_hits_step()` for determining appropriate time bucket sizes, and duration parsing with regex patterns.

### Health Monitoring
Basic health check functionality via the `ready()` method and corresponding `health` CLI command for deployment verification.

## Data Flows

```mermaid
graph TD
    A[CLI Command] --> B[Typer Argument Parser]
    B --> C[get_client()]
    C --> D[VictoriaLogsClient]
    D --> E[HTTP Request to VictoriaLogs]
    E --> F[JSON Response Processing]
    F --> G[Result Formatting]
    G --> H[Console Output]
    
    subgraph "Client Operations"
        D --> I[Query Building]
        I --> J[URL Construction]
        J --> K[Request Execution]
        K --> L[Response Validation]
    end
```

```mermaid
graph LR
    A[LogsQL Query] --> B[VictoriaLogsClient.query()]
    B --> C[POST /select/logsql/query]
    C --> D[Line-by-Line JSON Parsing]
    D --> E[Result List]
    
    F[Field Discovery] --> G[VictoriaLogsClient.field_names()]
    G --> H[POST /select/logsql/field_names]
    H --> I[Field Name Extraction]
    
    J[Analytics Query] --> K[Specialized Method]
    K --> L[Multiple API Calls]
    L --> M[Data Aggregation]
    M --> N[Statistics Generation]
```

## External Dependencies

### Runtime Libraries

- **httpx** (>=0.27.0) [networking]: Modern async-capable HTTP client library for making requests to VictoriaLogs API. Used throughout the VictoriaLogsClient class for all HTTP interactions. Imported in: `tools/infra/vlogs/client.py`.

- **typer** (>=0.12.0) [cli]: Command-line interface framework providing argument parsing, help generation, and command registration. Used to build the main CLI application with subcommands. Imported in: `tools/infra/vlogs/cli.py`.

- **rich** (>=13.0.0) [cli]: Terminal formatting library providing colored output, tables, and progress bars. Used for formatting CLI output and creating tables for streams display. Imported in: `tools/infra/vlogs/cli.py`.

- **python-dotenv** (>=1.0.0) [configuration]: Environment variable loading from .env files. Used to load configuration before starting the CLI application. Imported in: `tools/infra/vlogs/cli.py`.

### Standard Library Usage

- **json**: JSON serialization/deserialization for API responses and CLI output formatting
- **os**: Environment variable access for VictoriaLogs URL configuration  
- **re**: Regular expression matching for duration parsing and LogsQL value validation
- **typing**: Type annotations for function parameters and return values
- **collections**: defaultdict and Counter for analytics data aggregation
- **pathlib**: File path handling in test utilities

## API Surface

### CLI Commands

The component exposes a `vlogs` command-line tool with the following subcommands:

- **vlogs query** `<query>`: Execute LogsQL queries with optional time range and limit parameters
- **vlogs fields**: List all known field names, optionally filtered by query and time range  
- **vlogs field-values** `<field>`: Enumerate all values for a specific field
- **vlogs streams**: Discover log streams matching a query pattern
- **vlogs health**: Check VictoriaLogs service readiness

### Python API

The `VictoriaLogsClient` class provides programmatic access to VictoriaLogs functionality:

**Core Query Methods:**
- `query(query, limit, start, end)`: Execute LogsQL queries
- `hits(query, start, end, step)`: Get log hit statistics over time ranges
- `field_names(query, start, end)`: List available log field names  
- `field_values(field, query, limit, start, end)`: Get values for specific fields
- `streams(query, start, end)`: Find log streams

**Analytics Methods:**
- `thread_logs(thread_key, level, limit, start, end)`: Get logs for specific thread
- `thread_trace(thread_key, start, limit)`: End-to-end thread timeline
- `tool_analytics(start, limit)`: Tool usage statistics and performance metrics
- `execution_summaries(start, harness, status, prompt_ref, limit)`: Execution outcome analysis
- `service_health(start)`: Per-service error rates and log volumes

## External Systems

### VictoriaLogs Database
The component connects to a VictoriaLogs instance for log storage and querying. The connection URL is configurable via the `VICTORIALOGS_URL` environment variable, defaulting to `http://victorialogs:9428` for containerized deployments.

**API Endpoints Used:**
- `POST /select/logsql/query`: Execute LogsQL queries
- `POST /select/logsql/hits`: Get log hit statistics  
- `POST /select/logsql/field_names`: List field names
- `POST /select/logsql/field_values`: Get field values
- `POST /select/logsql/streams`: Find log streams
- `GET /health`: Health check endpoint
