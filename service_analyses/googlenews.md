# Google News RSS Library Analysis

## Overview

The googlenews component is a Python library that provides a programmatic interface to Google News RSS feeds. It acts as both a client library and CLI tool for searching news articles, retrieving headlines, and fetching topic-based news content. The library wraps Google's publicly available RSS endpoints without requiring API keys or authentication.

## Architecture

The component follows a simple layered architecture with clear separation between the client logic and CLI interface:

- **Client Layer** (`client.py`): Contains the core RSS client with HTTP fetching and parsing logic
- **CLI Layer** (`cli.py`): Provides command-line interface with multiple output formats
- **Entry Point** (`__init__.py`): Minimal module definition

The architecture pattern is straightforward functional with object-oriented client design, focusing on simplicity and ease of use.

## Key Components

### 1. GoogleNewsClient Class
The main client class provides three primary methods for accessing Google News RSS feeds:

- **`search(query, limit)`**: Searches for news articles using query terms
- **`headlines(country, limit)`**: Retrieves top headlines for a specific country
- **`topic(topic, country, limit)`**: Fetches news articles by predefined topics

### 2. RSS Feed Parser
The `_fetch_feed()` method handles the low-level RSS parsing using feedparser, extracting structured data including title, link, publication date, and source information.

### 3. CLI Command Interface
Three Typer-based commands mirror the client methods:
- **`search`**: Command-line search functionality
- **`headlines`**: CLI headlines retrieval  
- **`topic`**: Topic-based news fetching

### 4. Output Formatters
Multiple output format support including:
- **Rich Tables**: Styled terminal output with column formatting
- **Markdown Tables**: Plain text table format
- **JSON**: Machine-readable structured output

### 5. Utility Functions
Helper functions for text processing and client instantiation:
- **`truncate()`**: Text truncation for display formatting
- **`print_markdown_table()`**: Markdown table rendering
- **`get_client()`**: Client factory function

## Data Flows

```mermaid
graph TD
    A[CLI Command] --> B[get_client()]
    B --> C[GoogleNewsClient]
    C --> D[_fetch_feed()]
    D --> E[feedparser.parse()]
    E --> F[Google News RSS]
    F --> G[Raw RSS Data]
    G --> H[Structured Articles]
    H --> I{Output Format}
    I --> J[Rich Table]
    I --> K[Markdown Table]
    I --> L[JSON Output]
```

### Primary Data Flow
1. **Command Invocation**: User executes CLI command with parameters
2. **Client Creation**: Factory function instantiates GoogleNewsClient
3. **URL Construction**: Method builds appropriate Google News RSS URL
4. **RSS Fetching**: feedparser retrieves and parses RSS feed
5. **Data Extraction**: Raw RSS entries converted to structured dictionaries
6. **Formatting**: Results formatted according to user preference
7. **Output**: Final display via console, markdown, or JSON

### Error Handling Flow
1. **Feed Validation**: Check for feedparser bozo errors
2. **Exception Handling**: RuntimeError raised for fetch failures
3. **Empty Results**: Graceful handling with user messaging
4. **Typer Exit**: Clean CLI termination on errors

## External Dependencies

### Runtime Dependencies

- **feedparser** (>=6.0.0) [data-parsing]: Universal RSS/Atom feed parser that handles the core functionality of retrieving and parsing Google News RSS feeds. Used extensively in `_fetch_feed()` method for converting RSS XML to Python dictionaries. Imported in: `client.py`.

- **typer** (>=0.12.0) [cli]: Modern CLI framework built on top of Click, providing the command-line interface with argument parsing, help generation, and command routing. Powers all three CLI commands (search, headlines, topic) with options and arguments. Imported in: `cli.py`.

- **rich** (>=13.0.0) [cli]: Library for rich text and beautiful formatting in the terminal, specifically used for creating styled tables with columns, colors, and formatting. Used via `Console` and `Table` classes for the default output format. Imported in: `cli.py`.

### Development Dependencies

- **python-dotenv** [config]: Environment variable loading from .env files, though no secrets are actually required for this component. Imported in: `cli.py`.

### Internal Dependencies

- **centaur_sdk** [framework]: Internal SDK providing the `Table` class for formatted output, imported in `cli.py` at line 12.

## API Surface

### Library Interface (client.py)

```python
class GoogleNewsClient:
    def __init__(self, timeout: float = 30.0)
    def search(self, query: str, limit: int = 20) -> list[dict]
    def headlines(self, country: str = "US", limit: int = 20) -> list[dict] 
    def topic(self, topic: str, country: str = "US", limit: int = 20) -> list[dict]

def _client() -> GoogleNewsClient  # Factory function
```

### CLI Interface (cli.py)

```bash
googlenews search <query> [--limit N] [--json] [--markdown]
googlenews headlines [--country CODE] [--limit N] [--json] [--markdown]
googlenews topic <TOPIC> [--country CODE] [--limit N] [--json] [--markdown]
```

### Article Data Structure

All methods return lists of dictionaries with the following structure:
```python
{
    "title": str,      # Article headline
    "link": str,       # Article URL  
    "published": str,  # Publication timestamp
    "source": str      # News source name
}
```

The library provides a clean, consistent interface for accessing Google News content programmatically, suitable for integration into larger news monitoring, research, or content aggregation systems.
