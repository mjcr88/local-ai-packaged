# Ardoq MCP Server for Claude

## Overview

This project provides a Model Context Protocol (MCP) server that enables AI assistants like Claude to interact with the Ardoq Enterprise Architecture platform. It exposes functionalities of the Ardoq REST API (v2) as tools, allowing for natural language interaction with your Ardoq data models.

Built with Node.js and the `@modelcontextprotocol/sdk`.

## Features (Tools Provided)

The server offers a range of tools, grouped by functionality:

- **Core (`index.js`)**
    
    - `verify-connection`: Checks API token validity and connection details.
- **Basic CRUD (`basicCrudTools.js`)**
    
    - Workspaces: List, get, get context, create.
    - Components: List/filter (incl. custom fields), get, create, update, delete. (Includes optimistic locking).
    - References: List/filter, get, create, delete. (Includes optimistic locking).
    - Custom Fields: List definitions, update values on components.
- **Workspace Management (`workspaceMgmtTools.js`)**
    
    - Stats & Info: Get usage, aggregated data, overview, inventory.
    - Features: Toggle Gremlin, get change digest, initiate export, copy (v1 & v2).
    - Metamodel: Get/Update structure for a specific workspace (use cautiously).
    - Versioning: Get specific workspace versions.
    - Mutations: Rename, replace, delete multiple workspaces (includes optimistic locking).
- **Metamodel Containers (`metamodelTools.js`)**
    
    - List/Get/Create/Update/Delete Ardoq "Models". (Includes optimistic locking).
    - Rename Models, get specific versions.
    - _Note: Manages the container, not the types/fields within._
- **Reporting (`reportingTools.js`)**
    
    - Management: Create, copy, update, delete, rename reports. (Includes optimistic locking).
    - Search & Query: Search across workspaces, get aggregates, search within reports, drilldown.
    - Export: Export report data to JSON or Excel.
    - Utilities: Get aggregates for specific reports, get filter suggestions, check usage.
- **Intelligent Reporting (`intelligentReporting.js`)**
    
    - Analyzes report data to generate: Executive Summaries, Trend Analyses, Pattern/Anomaly Insights, Natural Language Interpretations. _(Note: Uses heuristics)._
- **NLP (`nlpComponentCreation.js`, `nlpQueryParser.js`)**
    
    - Metamodel Suggestion: Suggest types based on text descriptions.
    - Parsing: Extract components/relationships from text.
    - Creation: Create components/relationships from text (with validation).
    - Type Suggestion: Suggest component types for descriptions.
    - Query Parsing (Experimental): Basic parsing of NL queries to filters. _(Note: Basic implementation)._
- **Search (`graphSearch.js`, `advancedSearch.js`)**
    
    - `graph-search-nlp`: Basic NLP to Gremlin conversion for simple property searches via `POST /graph-search`. _(Workaround)_.
    - `graph-search-raw`: Execute raw Gremlin queries via `POST /graph-search`. _(Workaround)_.
    - `search-aggregations`: Get component aggregations (counts, terms, stats) via `GET /components/aggregations`.
- **Disabled Tools**
    
    - `advanced-search`: The tool using `POST /advanced-search` is currently disabled due to triggering internal SDK errors during tool listing. See Development Notes.

## Project Structure

```text
.
├── index.js              # Main server entry point
├── src/
│   ├── schemas.js        # Shared Zod schemas
│   └── tools/            # Tool modules
│       ├── advancedSearch.js # (Contains disabled tool code)
│       ├── basicCrudTools.js
│       ├── graphSearch.js
│       ├── intelligentReporting.js
│       ├── metamodelTools.js
│       ├── nlpComponentCreation.js
│       ├── nlpQueryParser.js
│       ├── reportingTools.js
│       └── workspaceMgmtTools.js
├── .env                  # Local environment variables (Not committed)
├── .env.example          # Example environment file
├── package.json
├── package-lock.json
├── README.md             # This file
└── .gitignore
```

## Getting Started

### Prerequisites

- Node.js (v18 or higher recommended)
- npm (usually included with Node.js)
- Ardoq instance access and an API Token

### Installation

1. Clone the repository:
    
    ```bash
    git clone <repository-url>
    cd ardoq-mcp
    ```
    
2. Install dependencies:
    
    ```bash
    npm install
    ```
    

### Configuration

1. Copy the example environment file:
    
    ```bash
    cp .env.example .env
    ```
    
2. Edit the new `.env` file with your Ardoq details:
    
    ```dotenv
    # .env - Configuration for Ardoq MCP Server
    
    # Required: Ardoq API Token
    # Generate from Ardoq: Preferences -> API keys and Scopes
    ARDOQ_API_TOKEN="YOUR_ARDOQ_API_TOKEN_HERE"
    
    # Required: Base URL for your Ardoq instance
    # Example: https://app.ardoq.com or https://your-company.ardoq.com
    # Do NOT include /api/v2 here.
    ARDOQ_API_URL="https://your-instance.ardoq.com"
    
    # Optional: Specify the Ardoq organization context if needed
    # Usually your org's subdomain/label shown in the Ardoq URL or UI
    # ARDOQ_ORG="your-organization-label"
    ```
    

## Running the Server

Start the server from the project root directory using Node.js:

```bash
node index.js
```

The server connects via stdio. Logs and `console.error` output will appear in the environment running the MCP server (e.g., Claude's tool execution console).

_Note: Remember to manage which tools are active by commenting/uncommenting the `init...Tools` calls in `index.js`._

## Development Notes

- **Refactoring:** This codebase includes centralized API requests, standard error handling, shared Zod schemas, ES Modules, and optimistic locking.
    
- **Known Issue - `advanced-search` Tool Disabled:** The `advanced-search` tool (using `POST /advanced-search`) is currently **disabled** in `index.js` due to triggering an internal SDK error (`Cannot read properties of null (reading '_def')`) during tool listing.
    
    - **Troubleshooting Summary:** Extensive debugging pointed towards a potential SDK v1.8.0 limitation or bug related to introspecting complex/nested Zod schemas when the full set of tools is registered. (Tested with Node v22.14.0, Zod v3.23.8).
    - **Workaround:** The `graph-search-nlp` and `graph-search-raw` tools provide alternative search functionality using Gremlin.
    - **Help Wanted:** Contributions to diagnose/fix the SDK interaction or enhance Gremlin generation are welcome. Consider reporting this to MCP maintainers with a reproducible example.
- **NLP:** Current NLP features are basic and may require enhancement for complex inputs.