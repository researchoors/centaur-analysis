# Attio CRM CLI Component Analysis

## Architecture

The attio component follows a clean layered architecture pattern with clear separation between the CLI interface and the API client. The component is structured as:

1. **CLI Layer** (`cli.py`): Typer-based command interface with rich formatting
2. **Client Layer** (`client.py`): HTTP API client wrapper for Attio CRM REST API
3. **Configuration**: Environment-based authentication via dotenv

The architecture employs a facade pattern where the CLI commands provide user-friendly interfaces to complex CRM operations, while the client handles all HTTP communication, authentication, and error handling.

## Key Components

### AttioClient Class
The core API client that manages authenticated requests to Attio's REST API at `https://api.attio.com/v2`. Features lazy HTTP client initialization, automatic JSON parsing, and comprehensive error handling with status code checking.

### CLI Command Structure
Eighteen distinct commands organized by CRM entity types:
- **Metadata commands**: `whoami`, `objects`, `attributes`, `members`
- **Record operations**: `people`, `companies`, `records`, `get`, `create`, `update`, `delete`, `upsert`  
- **List management**: `lists`, `entries`
- **Activity tracking**: `notes`, `add-note`, `tasks`, `add-task`

### Value Extraction System
A sophisticated `_extract_value()` function that handles Attio's complex nested value objects, supporting multiple field types (names, emails, domains, phone numbers) with fallback hierarchies.

### Authentication Manager
Secure credential handling using the centaur_sdk secret management system with environment variable fallback and runtime validation.

### Rich Formatting Engine
Consistent table-based output using the centaur_sdk Table component with color coding, column sizing, and truncation for optimal CLI experience.

## Data Flows

```mermaid
graph TD
    A[CLI Command] --> B[_get_client()]
    B --> C[AttioClient.__init__]
    C --> D[Load .env secrets]
    D --> E[Lazy HTTP client init]
    A --> F[Client API method]
    F --> G[_request() wrapper]
    G --> H[httpx HTTP call]
    H --> I[Attio REST API]
    I --> J[JSON response parsing]
    J --> K[Error handling]
    K --> L[Rich table formatting]
    L --> M[Console output]
```

### Record Query Flow
1. User invokes CLI command with filters/parameters
2. CLI validates and transforms parameters to Attio API format
3. Client constructs POST request with filter objects and pagination
4. API returns paginated JSON results with nested value structures
5. Value extraction flattens complex objects for display
6. Rich tables present formatted data with ID truncation and styling

### CRUD Operation Flow  
1. CLI commands parse JSON input for record values
2. Client validates required authentication credentials
3. HTTP requests sent with proper Content-Type and Authorization headers
4. API responses include full record objects with generated IDs
5. Success/error feedback provided via colored console output

## External Dependencies

### External Libraries

- **httpx** (>=0.27.0) [networking]: Modern async-capable HTTP client library providing the core API communication layer. Used extensively in `AttioClient._http()` for creating authenticated sessions with base URL, headers, and timeout configuration. Imported in: `tools/business/attio/client.py`.

- **typer** (>=0.12.0) [cli]: CLI framework built on Click providing command parsing, argument validation, and help generation. Powers all 18 CLI commands with decorators like `@app.command()` and handles option/argument parsing. Imported in: `tools/business/attio/cli.py`.

- **rich** (>=13.0.0) [cli]: Advanced terminal formatting library providing colored output, tables, and console styling. Used for creating formatted tables via `Console()` and `Table()` classes for all data display commands. Imported in: `tools/business/attio/cli.py`.

- **python-dotenv** (>=1.0.0) [config]: Environment variable management from `.env` files. Loaded at module level via `load_dotenv()` to support local development credential configuration. Imported in: `tools/business/attio/cli.py`.

## API Surface

### CLI Commands

The component exposes 18 commands through the `attio` executable:

**Authentication & Discovery**
- `whoami`: Display workspace information for current API token
- `objects`: List all CRM objects (people, companies, custom objects) 
- `attributes`: Show field definitions for any object type
- `members`: List workspace members with roles and contact info

**Record Management**
- `people`: Query people records with name/email filtering
- `companies`: Query company records with name/domain filtering  
- `records`: Generic record querying for any object type
- `get`: Retrieve single record by ID
- `create`: Create new records with JSON value specification
- `update`: Modify existing records via JSON patches
- `delete`: Remove records with confirmation prompts
- `upsert`: Create or update based on matching attributes

**List Operations**
- `lists`: Display all lists in workspace
- `entries`: Query entries within specific lists

**Activity Tracking**
- `notes`: List notes attached to records
- `add-note`: Create new notes with title and content
- `tasks`: Query tasks with filtering by completion status
- `add-task`: Create tasks with deadlines and assignees

### Python Client API

The `AttioClient` class provides programmatic access with 25+ methods covering:

**Object & Schema APIs**: `list_objects()`, `get_object()`, `list_attributes()`

**Record CRUD**: `query_records()`, `get_record()`, `create_record()`, `update_record()`, `delete_record()`, `assert_record()`

**List Management**: `list_lists()`, `get_list()`, `query_entries()`, `create_entry()`

**Notes & Tasks**: `list_notes()`, `create_note()`, `list_tasks()`, `create_task()`

**Advanced Features**: `list_meetings()`, `get_meeting()`, `list_call_recordings()`, `get_call_transcript()`, `list_threads()`, `get_thread()`

**Workspace Management**: `list_workspace_members()`, `get_self()`

**Raw API Access**: `raw_request()` for direct REST API calls

All methods return parsed JSON dictionaries and handle authentication, error responses, and API-specific parameter encoding automatically.
