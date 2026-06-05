# PostHog CLI Component Analysis

The PostHog component is a command-line interface tool for interacting with PostHog's product analytics platform. It provides a Python-based CLI that executes HogQL queries and generates formatted reports for web analytics data.

## Architecture

The component follows a classic CLI architecture with clear separation between the command-line interface and the API client. It uses a **client-server pattern** where the local CLI acts as a client to the remote PostHog API service. The architecture consists of:

- **CLI Layer** (`cli.py`): Command definitions using Typer framework
- **Client Layer** (`client.py`): HTTP API client with authentication and query execution
- **Configuration**: Environment-based configuration for API credentials

The design emphasizes clean separation of concerns with the CLI handling user interaction and formatting while the client manages all API communication and HogQL query construction.

## Key Components

### PostHogClient Class
The core API client that handles authentication and HTTP requests to PostHog's REST API. It supports HogQL queries and provides pre-built analytics methods for common use cases like pageviews and breakdowns.

### CLI Command Handlers
Five main commands that provide different analytics views:
- `query`: Execute arbitrary HogQL queries
- `breakdown`: Property-based event breakdowns (browser, OS, etc.)
- `pageviews`: Website traffic analytics
- `user-agents`: Browser and OS distribution analysis
- `events`: Raw event listing with filtering

### Authentication System
Secure credential management using the centaur-sdk secret system, supporting both environment variables and runtime configuration for API keys and project IDs.

### Output Formatters
Multiple output formats including rich console tables, markdown tables, and JSON output to support different use cases from interactive exploration to automation and reporting.

### HTTP Client Management
Proper resource management with context manager support and configurable timeouts for reliable API communication.

## Data Flows

```mermaid
flowchart TD
    A[CLI Command] --> B[Argument Parsing]
    B --> C[get_client()]
    C --> D[PostHogClient.__init__]
    D --> E[Load Credentials]
    E --> F[Execute Method]
    F --> G[Build HogQL Query]
    G --> H[HTTP Request to PostHog API]
    H --> I[Parse JSON Response]
    I --> J[Format Output]
    J --> K[Display Results]
    
    E --> E1[Check Instance Variables]
    E1 --> E2[Check Environment Variables]
    E2 --> E3[Check centaur-sdk Secrets]
    
    J --> J1[Rich Table]
    J --> J2[Markdown Table]
    J --> J3[JSON Output]
```

### Query Execution Flow
1. User invokes CLI command with parameters
2. CLI validates arguments and creates PostHogClient instance
3. Client loads API credentials from environment or centaur-sdk
4. Method-specific HogQL query is constructed with user filters
5. HTTP POST request sent to PostHog API with authentication
6. JSON response parsed and formatted according to output preference
7. Results displayed to user via Rich console or printed as markdown/JSON

### Error Handling Flow
The component implements comprehensive error handling at multiple levels:
- Network errors caught and converted to user-friendly messages
- API authentication failures detected and reported
- Missing configuration gracefully handled with clear error messages
- HTTP status errors converted to RuntimeError with context

## External Dependencies

### External Libraries

- **httpx** (>=0.27.0) [networking]: Modern HTTP client library providing async/sync support and comprehensive error handling. Used in `PostHogClient._request()` for all API communication to PostHog services. Imported in: `client.py`.

- **typer** (>=0.12.0) [cli]: Command-line interface framework built on Click that provides type hints and automatic help generation. Powers all CLI commands including `query`, `breakdown`, `pageviews`, `user-agents`, and `events`. Imported in: `cli.py`.

- **rich** (>=13.0.0) [cli]: Rich text and beautiful formatting library for terminal output. Used via `Console` for colored error messages and `Table` (from centaur-sdk) for formatted analytics output. Imported in: `cli.py`.

- **python-dotenv** (>=1.0.0) [cli]: Environment variable loading from .env files for development configuration. Listed as dependency but not directly imported, likely used by the build system for development environment setup.

The component also depends on the internal `centaur_sdk` module for:
- `secret()` function for secure credential management
- `Table` class for rich table formatting (wraps rich.table.Table)

## API Surface

The component exposes both a programmatic API through the PostHogClient class and a command-line interface.

### Command-Line Interface

**posthog query** - Execute arbitrary HogQL queries
```bash
posthog query "SELECT event, count() FROM events GROUP BY event"
```

**posthog breakdown** - Property-based event analysis
```bash
posthog breakdown --property=$browser --event=page_view --days=30
```

**posthog pageviews** - Website traffic analytics
```bash
posthog pageviews --url="/dashboard" --days=7
```

**posthog user-agents** - Browser/OS distribution
```bash
posthog user-agents --days=14 --limit=50
```

**posthog events** - Raw event listing
```bash
posthog events --after=2024-01-01 --before=2024-01-31
```

All commands support `--json`, `--markdown`, and rich console output formats.

### Programmatic API

**PostHogClient** provides methods for:
- `query(sql, name=None)`: Execute arbitrary HogQL queries
- `events()`, `breakdown()`, `pageviews()`, `user_agents()`: Pre-built analytics queries
- Context manager support for proper resource cleanup
- Configurable authentication and host settings

## External Systems

The component integrates with the following external systems:

**PostHog Cloud API** (us.posthog.com by default)
- REST API endpoint at `/api/projects/{project_id}/query/`
- Requires Bearer token authentication with personal API key
- Accepts HogQL queries in JSON payloads
- Returns structured analytics data with columns and results arrays

**Configuration Sources**
- Environment variables: `POSTHOG_API_KEY`, `POSTHOG_PROJECT_ID`, `POSTHOG_HOST`
- centaur-sdk secret management system for secure credential storage
- Optional `.env` file support for development environments

## Component Interactions

This component has no internal dependencies on other centaur-src components. It operates independently as a standalone CLI tool that connects directly to external PostHog services. The only internal dependency is on the `centaur_sdk` module for secret management and UI components.
