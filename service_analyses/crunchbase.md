# Crunchbase Enterprise API Client

## Architecture

The crunchbase component is a Python library implementing a comprehensive client for the Crunchbase Enterprise API v4. It follows a **layered architecture** with clear separation between the HTTP client layer (`client.py`) and the CLI interface layer (`cli.py`). The design emphasizes ease of use through field aliasing, robust error handling, and multiple output formats (JSON, rich tables, and markdown).

The component is structured around two main layers:
- **API Client Layer**: Handles HTTP communication, authentication, and data normalization
- **CLI Interface Layer**: Provides command-line access with rich formatting and user-friendly features

## Key Components

### CrunchbaseClient Class
The core HTTP client that handles all API communication with Crunchbase's Enterprise API. It provides methods for entity lookups, searches, and autocomplete functionality. Features include field name aliasing to handle common LLM-generated mistakes, automatic API key management through the centaur SDK, and comprehensive error handling with meaningful messages.

### CLI Commands
Eight primary commands provide comprehensive access to Crunchbase data:
- `org`: Organization lookup with support for cards and custom fields
- `person`: Person entity lookup
- `funding`: Funding round details
- `card`: Paginated organization card data
- `search`: Multi-collection search with filtering
- `autocomplete`: Entity name completion
- `recent_funding`: Specialized query for recent large funding rounds
- `raw`: Direct API endpoint access

### Data Formatting Utilities
Specialized formatting functions handle Crunchbase's complex nested data structures:
- `format_money()`: Converts monetary values to human-readable formats (M, B suffixes)
- `extract_identifier()`: Extracts names from Crunchbase identifier objects
- `extract_identifiers()`: Handles lists of identifiers with truncation
- `extract_locations()`: Formats location data from location identifiers

### Field Aliasing System
A mapping system (`_FIELD_ALIASES`) corrects common field name mistakes, particularly those made by LLMs. This includes converting "lead_investors" to "lead_investor_identifiers" and similar pluralization corrections.

## Data Flows

```mermaid
graph TD
    A[CLI Command] --> B[get_client()]
    B --> C[CrunchbaseClient]
    C --> D[_get_api_key()]
    D --> E[centaur_sdk.secret]
    C --> F[_normalize_field_ids()]
    F --> G[_request()]
    G --> H[httpx.Client]
    H --> I[Crunchbase API v4]
    I --> J[JSON Response]
    J --> K[Format Output]
    K --> L[Console/JSON/Markdown]
```

### Request Processing Flow
1. CLI command receives user input and parses arguments
2. Field IDs are normalized through the aliasing system
3. API client constructs HTTP request with proper headers and authentication
4. Response data flows through formatting utilities based on output format
5. Results are presented via Rich console, JSON, or markdown formats

### Search and Pagination Flow
Complex queries and large result sets are handled through POST endpoints with JSON bodies containing query predicates, ordering specifications, and pagination cursors (`after_id`).

## External Dependencies

### External Libraries

- **httpx** (>=0.27.0) [networking]: Modern HTTP client library providing async/sync support. Used as the primary HTTP transport for all API requests to Crunchbase. The client is instantiated lazily and supports connection pooling and timeout configuration. Imported in: `tools/research/crunchbase/client.py`.

- **typer** (>=0.12.0) [cli]: Command-line interface framework built on Click. Provides argument parsing, help generation, and command routing for all CLI commands. Used extensively for decorators like `@app.command()` and parameter definitions with `typer.Argument()` and `typer.Option()`. Imported in: `tools/research/crunchbase/cli.py`.

- **rich** (>=13.0.0) [cli]: Advanced terminal formatting library providing colored output, tables, and progress indicators. Used for creating formatted tables via `Table()` class and console output via `Console()`. Enables rich text formatting with markup like `[bold cyan]` and `[yellow]`. Imported in: `tools/research/crunchbase/cli.py`.

### Internal Dependencies

- **centaur_sdk** [internal]: Internal SDK providing the `secret()` function for secure credential management. Used to retrieve the `CRUNCHBASE_API_KEY` from environment variables or secure storage. This is the only internal dependency, imported in: `tools/research/crunchbase/client.py`.

## API Surface

The component exposes its functionality through two primary interfaces:

### Python Library Interface
- **CrunchbaseClient**: Main client class with methods for all Crunchbase entity types
- Entity lookup methods: `get_organization()`, `get_person()`, `get_funding_round()`, etc.
- Search methods: `search_organizations()`, `search_people()`, `search_funding_rounds()`, etc.
- Utility methods: `autocomplete()`, `get_deleted_entities()`, `raw()`
- Context manager support for resource cleanup

### CLI Interface
- **crunchbase** command with 8 subcommands
- Multiple output formats: default rich tables, `--json`, `--markdown`
- Field selection via `--fields` parameter
- Pagination support with `--after` cursors
- Advanced filtering through JSON query syntax

### Configuration
- API key management through `CRUNCHBASE_API_KEY` environment variable
- Configurable timeout settings (default 30 seconds)
- Field aliasing system for user-friendly field names

## External Systems

### Crunchbase Enterprise API v4
The component integrates with Crunchbase's Enterprise API hosted at `api.crunchbase.com`. Authentication uses API key headers (`X-cb-user-key`). The API provides comprehensive startup, investor, and funding data through REST endpoints supporting both GET and POST methods for complex queries.

### Configuration and Secrets
Relies on the centaur SDK's secret management system for secure API key storage and retrieval. The API key must be configured as `CRUNCHBASE_API_KEY` in the environment.

## Component Interactions

This component has no dependencies on other components in the centaur codebase beyond the shared `centaur_sdk` for secret management. It is designed as a standalone library that can be consumed by other components or used directly via its CLI interface.
