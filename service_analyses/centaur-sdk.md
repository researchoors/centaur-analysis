# Centaur SDK Analysis

## Architecture

The Centaur SDK follows a layered architecture pattern with distinct concerns:

1. **Public API Layer**: Clean interface exposed through `__init__.py` for secret management, tool context, and table rendering
2. **Secret Backend System**: Pluggable architecture for secret resolution with multiple backend implementations
3. **Tool Context Management**: Thread-safe context variables for managing tool execution state
4. **Utility Modules**: Supporting functionality for logging and CLI table rendering

The design emphasizes security through secret isolation, with different backends for CLI vs server environments to prevent secret leakage.

## Key Components

### 1. **Tool SDK Core** (`tool_sdk.py`)
The primary module containing the main SDK functionality:
- `ToolContext` dataclass storing tool name, secrets, thread key, and container ID
- Context variable management using Python's `contextvars` for thread-safe tool isolation
- Secret resolution with fallback chain: tool context → backend → default
- Attachment management for uploading files to Centaur API

### 2. **Secret Backend System** (`backends/`)
Pluggable architecture for secret management:
- `SecretBackend` abstract base class defining async/sync interface
- `StubBackend` for server mode returning key names as placeholders for firewall injection
- `EnvBackend` for CLI mode reading from environment variables
- `Registry` module managing the global backend singleton

### 3. **Logging Infrastructure** (`logging.py`)
Structured JSON logging implementation:
- `JsonFormatter` class producing single-line JSON logs
- Service-specific configuration with timestamp, level, and event fields
- Support for uvicorn integration in web services

### 4. **CLI Table Utilities** (`cli_tables.py`)
Table rendering helpers for command-line tools:
- Re-export of Rich library's Table class
- Plain-text table renderer with automatic column width calculation

### 5. **Test Suite** (`tests/test_tool_sdk.py`)
Comprehensive unit tests covering:
- Secret resolution priority ordering
- Backend behavior verification
- Context management and thread isolation
- Synchronous/asynchronous execution patterns

## Data Flows

```mermaid
graph TD
    A[Tool Code] --> B[secret() function]
    B --> C{Tool Context Available?}
    C -->|Yes| D[Check Context Secrets]
    C -->|No| E[Query Backend]
    D --> F{Key Found?}
    F -->|Yes| G[Return Value]
    F -->|No| E
    E --> H{Backend Has Key?}
    H -->|Yes| G
    H -->|No| I{Default Provided?}
    I -->|Yes| G
    I -->|No| J[Raise KeyError]
```

```mermaid
graph TD
    A[save_attachment()] --> B[Get Thread Key]
    B --> C[Prepare Payload]
    C --> D[Base64 Encode Data]
    D --> E[Add API Key Header]
    E --> F[HTTP POST to Centaur API]
    F --> G[Parse Response]
    G --> H[Return Attachment Metadata]
```

## External Dependencies

### External Libraries

- **rich** (>=13.0) [ui]: Provides Rich text rendering capabilities including the Table class for CLI formatting. Used in `cli_tables.py` as a re-export for tool developers. Imported in: `centaur_sdk/cli_tables.py`.

### Optional Dependencies

- **httpx** (>=0.28.0) [networking]: Optional HTTP client library listed in optional dependencies but not currently used in the codebase. The SDK uses Python's built-in `urllib.request` for HTTP operations instead.

### Standard Library Dependencies

The SDK extensively uses Python's standard library:
- **contextvars**: Thread-local storage for tool context management
- **urllib.request**: HTTP client for attachment uploads to Centaur API
- **json**: JSON serialization for logging and API communication
- **base64**: Binary data encoding for file attachments
- **mimetypes**: MIME type detection for uploaded files
- **logging**: Structured logging infrastructure
- **asyncio**: Asynchronous execution support in secret backends

## API Surface

### Primary Functions

- `secret(key, default=None)`: Resolve secrets with configurable fallback chain
- `save_attachment(name, data, mime_type=None, source_url=None)`: Upload binary data as thread-scoped attachment
- `save_attachment_from_path(path, name=None, mime_type=None, source_url=None)`: Upload local files

### Context Management

- `get_tool_context()`: Access current tool execution context
- `set_tool_context(ctx)`: Establish tool context (returns reset token)
- `reset_tool_context(token)`: Restore previous context state
- `current_thread_key()`: Get active thread identifier for scoped operations

### Backend Configuration

- `configure(backend)`: Set active secret backend implementation
- `get_backend()`: Access current backend (auto-configures if needed)

### Utility Functions

- `Table`: Rich table class for CLI formatting (re-export)
- `render_text_table(headers, rows)`: Plain-text table with auto-sized columns
- `configure_json_logging(service_name, level=None, uvicorn=False)`: Structured logging setup

The SDK provides a clean separation between CLI and server modes through its backend system, ensuring secrets are handled appropriately for each environment while maintaining a consistent API surface.
