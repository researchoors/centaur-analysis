Now I have comprehensive understanding of the pylon component. Let me write the analysis.

# Pylon Component Analysis

## Overview

The pylon component is a Python library that provides a comprehensive CLI interface for interacting with the Pylon support platform. It enables AI agents and users to manage support issues, accounts, and contacts through a well-structured command-line interface backed by a REST API client.

## Architecture

The component follows a **layered architecture** with clear separation of concerns:

- **CLI Layer** (`cli.py`): Command-line interface using Typer framework for user interaction
- **Client Layer** (`client.py`): HTTP API client handling authentication and REST API communication  
- **Configuration Layer**: Environment-based configuration for API credentials

The architecture emphasizes modularity, with the client being reusable independently of the CLI, making it suitable for integration into other systems or direct programmatic use.

## Key Components

### 1. PylonClient Class (`client.py`)
The core API client that handles all HTTP communication with the Pylon REST API at `api.usepylon.com`. Provides methods for CRUD operations on issues, accounts, contacts, users, teams, and tags.

### 2. CLI Application (`cli.py`)  
A Typer-based command-line application with 15 commands organized into functional groups:
- **Organization**: `me` command for getting org details
- **Issues**: `issues`, `issue`, `issue-create`, `issue-update`, `issue-search` 
- **Accounts**: `accounts`, `account`, `account-create`
- **Contacts**: `contacts`, `contact`
- **Users & Teams**: `users`, `teams`
- **Tags**: `tags`

### 3. Authentication Handler
Centralized API key management using the centaur_sdk secret system, with fallback to environment variables and clear error messaging for missing credentials.

### 4. Request/Response Processing
Standardized HTTP request handling with proper error handling, timeout management, and JSON response parsing across all API endpoints.

### 5. Data Presentation Layer
Rich console formatting with tables for list views and detailed formatting for individual items, plus optional JSON output for programmatic consumption.

## Data Flows

```mermaid
graph TD
    A[CLI Command] --> B[Command Handler]
    B --> C[_get_client()]
    C --> D[PylonClient]
    D --> E[_get_api_key()]
    E --> F[centaur_sdk.secret()]
    D --> G[_request()]
    G --> H[httpx.Client]
    H --> I[Pylon API]
    I --> J[JSON Response]
    J --> K[Response Processing]
    K --> L[Rich Console Output]
```

### Primary Data Flow Patterns

1. **Command Execution Flow**: User command → CLI handler → Client method → HTTP request → API response → Formatted output
2. **Authentication Flow**: API call → Key retrieval → Bearer token header → Authenticated request
3. **Error Handling Flow**: API error → Exception parsing → User-friendly error message → Exit code

## External Dependencies

### External Libraries

- **httpx** (>=0.27.0) [networking]: Modern HTTP client for making async and sync HTTP requests to the Pylon API. Used in `PylonClient._request()` method for all API communication with timeout support and automatic connection management. Imported in: `tools/business/pylon/client.py`.

- **typer** (>=0.12.0) [cli]: Modern CLI framework for building command-line applications with automatic help generation and type validation. Provides the `@app.command()` decorators and argument/option parsing for all 15 CLI commands. Imported in: `tools/business/pylon/cli.py`.

- **rich** (>=13.0.0) [cli]: Library for rich text and beautiful formatting in the terminal. Used via `Console` class for colored output and error messages throughout the CLI. Imported in: `tools/business/pylon/cli.py`.

- **centaur_sdk** (implicit) [framework]: Centaur platform SDK providing `Table` class for tabular data display and `secret()` function for secure credential management. The `Table` class is used for formatting list outputs like issues, accounts, contacts. The `secret()` function retrieves the PYLON_API_KEY from the environment. Imported in: `tools/business/pylon/cli.py` and `tools/business/pylon/client.py`.

- **python-dotenv** (implicit via `load_dotenv()`) [configuration]: Loads environment variables from `.env` files. Called at module level in `cli.py` to ensure API keys are available from local development files. Imported in: `tools/business/pylon/cli.py`.

## API Surface

### Public Client Methods (PylonClient)

**Organization**
- `get_me()`: Get organization details for current API token

**Issues Management**
- `list_issues(start_time, end_time, days)`: List issues within time range
- `get_issue(issue_id)`: Get specific issue by ID/number
- `search_issues(filter_obj, limit, cursor)`: Search issues with filters
- `create_issue(title, body_html, **options)`: Create new issue
- `update_issue(issue_id, **updates)`: Update existing issue
- `delete_issue(issue_id)`: Delete issue

**Accounts Management**  
- `list_accounts(limit, cursor)`: List accounts with pagination
- `get_account(account_id)`: Get specific account
- `search_accounts(filter_obj, limit, cursor)`: Search accounts
- `create_account(name, **options)`: Create new account
- `update_account(account_id, **updates)`: Update account

**Contacts Management**
- `list_contacts()`: Get all contacts
- `get_contact(contact_id)`: Get specific contact  
- `search_contacts(filter_obj, limit, cursor)`: Search contacts
- `create_contact(name, email, account_id)`: Create new contact

**Users & Teams**
- `list_users()`: Get organization users
- `get_user(user_id)`: Get specific user
- `list_teams()`: Get organization teams
- `list_tags()`: Get all available tags

### CLI Commands

**Core Commands**
- `pylon me`: Show organization info
- `pylon issues [options]`: List/filter recent issues
- `pylon issue <id> [--json]`: Get issue details
- `pylon issue-create <title> <body> [options]`: Create issue
- `pylon issue-update <id> [options]`: Update issue  
- `pylon issue-search <filter> [options]`: Search issues
- `pylon accounts [options]`: List accounts
- `pylon account <id> [--json]`: Get account details
- `pylon account-create <name> [options]`: Create account
- `pylon contacts [options]`: List contacts
- `pylon contact <id> [--json]`: Get contact details
- `pylon users [options]`: List users
- `pylon teams`: List teams
- `pylon tags [--type]`: List tags

## External Systems

- **Pylon API** (api.usepylon.com): Primary external service providing REST API for support ticket management, account management, and contact management. All HTTP requests use Bearer token authentication with the PYLON_API_KEY.

## Component Interactions

The pylon component has no direct interactions with other components in the centaur-src codebase. It operates as a standalone library that depends only on the centaur_sdk for secret management and table formatting utilities.
