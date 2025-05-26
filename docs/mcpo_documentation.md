MCPO GitHub Documentation & Files


# ⚡️ mcpo

Expose any MCP tool as an OpenAPI-compatible HTTP server—instantly.

mcpo is a dead-simple proxy that takes an MCP server command and makes it accessible via standard RESTful OpenAPI, so your tools "just work" with LLM agents and apps expecting OpenAPI servers.

No custom protocol. No glue code. No hassle.

## 🤔 Why Use mcpo Instead of Native MCP?

MCP servers usually speak over raw stdio, which is:

- 🔓 Inherently insecure
- ❌ Incompatible with most tools
- 🧩 Missing standard features like docs, auth, error handling, etc.

mcpo solves all of that—without extra effort:

- ✅ Works instantly with OpenAPI tools, SDKs, and UIs
- 🛡 Adds security, stability, and scalability using trusted web standards
- 🧠 Auto-generates interactive docs for every tool, no config needed
- 🔌 Uses pure HTTP—no sockets, no glue code, no surprises

What feels like "one more step" is really fewer steps with better outcomes.

mcpo makes your AI tools usable, secure, and interoperable—right now, with zero hassle.

## 🚀 Quick Usage

We recommend using uv for lightning-fast startup and zero config.

```bash
uvx mcpo --port 8000 --api-key "top-secret" -- your_mcp_server_command
```

Or, if you’re using Python:

```bash
pip install mcpo
mcpo --port 8000 --api-key "top-secret" -- your_mcp_server_command
```

To use an SSE-compatible MCP server, simply specify the server type and endpoint:

```bash
mcpo --port 8000 --api-key "top-secret" --server-type "sse" -- http://127.0.0.1:8001/sse
```

You can also run mcpo via Docker with no installation:

```bash
docker run -p 8000:8000 ghcr.io/open-webui/mcpo:main --api-key "top-secret" -- your_mcp_server_command
```

Example:

```bash
uvx mcpo --port 8000 --api-key "top-secret" -- uvx mcp-server-time --local-timezone=America/New_York
```

That’s it. Your MCP tool is now available at http://localhost:8000 with a generated OpenAPI schema — test it live at [http://localhost:8000/docs](http://localhost:8000/docs).

🤝 **To integrate with Open WebUI after launching the server, check our [docs](https://docs.openwebui.com/openapi-servers/open-webui/).**

### 🔄 Using a Config File

You can serve multiple MCP tools via a single config file that follows the [Claude Desktop](https://modelcontextprotocol.io/quickstart/user) format:

Start via:

```bash
mcpo --config /path/to/config.json
```

Example config.json:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "time": {
      "command": "uvx",
      "args": ["mcp-server-time", "--local-timezone=America/New_York"]
    },
    "mcp_sse": {
      "url": "http://127.0.0.1:8001/sse"
    } // SSE MCP Server
  }
}
```

Each tool will be accessible under its own unique route, e.g.:
- http://localhost:8000/memory
- http://localhost:8000/time

Each with a dedicated OpenAPI schema and proxy handler. Access full schema UI at: `http://localhost:8000/<tool>/docs`  (e.g. /memory/docs, /time/docs)

## 🔧 Requirements

- Python 3.8+
- uv (optional, but highly recommended for performance + packaging)

## 🛠️ Development & Testing

To contribute or run tests locally:

1.  **Set up the environment:**
    ```bash
    # Clone the repository
    git clone https://github.com/open-webui/mcpo.git
    cd mcpo

    # Install dependencies (including dev dependencies)
    uv sync --dev
    ```

2.  **Run tests:**
    ```bash
    uv run pytest
    ```


## 🪪 License

MIT

## 🤝 Contributing

We welcome and strongly encourage contributions from the community!

Whether you're fixing a bug, adding features, improving documentation, or just sharing ideas—your input is incredibly valuable and helps make mcpo better for everyone.

Getting started is easy:

- Fork the repo
- Create a new branch
- Make your changes
- Open a pull request

Not sure where to start? Feel free to open an issue or ask a question—we’re happy to help you find a good first task.

## ✨ Star History

<a href="https://star-history.com/#open-webui/mcpo&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=open-webui/mcpo&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=open-webui/mcpo&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=open-webui/mcpo&type=Date" />
  </picture>
</a>

---

✨ Let's build the future of interoperable AI tooling together!



MCPO: Supercharge Open-WebUI /Ollama with MCP Tools

Minyang Chen
Follow
10 min read
·
Apr 4, 2025
247
2
Open WebUI officially supports MCP Tool Servers — as long as the MCP Tool Server is fronted by an OpenAPI-compatible proxy (openapi-servers) — a purpose-built proxy implementation available at: 👉 https://github.com/open-webui/mcpo.
After trying a local setup, experimenting with configurations, and testing prompts, I discovered that mcpo is a simple proxy that allows you to use MCP server commands with tools that work with standard OpenAPI servers. This makes it easy to connect your tools with LLM agents and apps.
In this article, we will first explore how it works, then create a brand new MCP server, launch it, and add it as a tool to open-webui.
MCPO Architecture

figure. MCPO architecture
According to the solution architecture, the MCPO interacts directly with the MCP servers using standard input/output (stdio) transport. Subsequently, all MCP communication is converted to a RESTful API when interacting with Open-WebUI.
Ready, let’s try it now!
Table of Content
Prerequisites
Configure MCPO Server
Configure Open-WebUI Tools
Test Tool-Calls
Creating Brand New MCP Server: Tariff News Reaction
Generate Tariff News Reaction MCP Server Code
Test New MCP Server tool call in Roo Code
Add New MCP Server to MCPO server configuration
Add New MCP Server to Open-WebUI
Test Tool-Call
Final Thoughts
Prerequisites
Download and install Ollama if your don’t have locally, see here: https://ollama.com/download
Download and Install open-webui if you don’t have locally, see here: https://github.com/open-webui/open-webui
NodeJS
Python 3.11 (open-webui required) / pip or using uv (curl -LsSf https://astral.sh/uv/install.sh | sh)
VS Code + Roo Code + Google Gemini 2.5 Pro (optional, use for code generation of new MCP server)
Configure MCPO Server
Create a new python virtual environment
$ python -m venv .venv
$ source .venv/bin/activate
2. Install MCPO server
$ pip install mcpo
3. Install MCP Servers
Select servers from here: https://github.com/modelcontextprotocol/servers. for now let try install following 3 servers, they are time, memory and fetch.
# 1.time mcp server
$ pip install mcp-server-time

# 2.memory mcp server
$ npm install @modelcontextprotocol/server-memory

# 3.fetch mcp server
$ pip install mcp-server-fetch
4. Next, create a config.json file so we can connect to multiple MCP server instances using a single MCPO server.
❯ cat config.json
{
 "mcpServers": {
   "memory": {
     "command": "npx",
     "args": ["-y", "@modelcontextprotocol/server-memory"]
   },
   "time": {
     "command": "uvx",
     "args": ["mcp-server-time", "--local-timezone=America/New_York"]
   },
 "fetch": {
   "command": "uvx",
   "args": ["mcp-server-fetch"]
 }
 }
}
5. Run the MCPO server
$ uvx mcpo --config config.json --port 8001


## result log
❯ uvx mcpo --config config.json --port 8001
Starting MCP OpenAPI Proxy with config file: config.json
INFO:     Started server process [1190222]
INFO:     Waiting for application startup.
Knowledge Graph MCP Server running on stdio
Okay, at this point, we complete the MCPO server setup.
6. Verify MCP Server with document link generated by MCPO.

figure. MCPO generated API docs for MCP tools
so far looks good. MCP server is up and running with one tool call fetch.
Configure Open-WebUI tools
Next, to add the MCPO endpoints as tools in Open-WebUI, click Settings > Tools > +. Then, enter the URL of the MCPO tool and click Save.
See screenshots below:

figure. Add Tools in Open-WebUI
To verify the tools are enable, we can click the [tools] icon in input chat window next to the [mic] icon.

figure. Open-WebUI with Tools enable
Test Tool Calls in Open-WebUI
Let’s try use the fetch tool by asking a question to retrieve content from a URL.

figure. Open-WebUI with Tools Call
We see a Tool_endpoint_fetch_post log, which indicates the response was generated using the tool-call feature – a powerful capability. So far, everything looks good. We’re confirming that Open-WebUI support for MCP Tools works with MCPO as a proxy.
Next, let’s conduct another experiment: we’ll create a new MCP server and add it to MCPO to see what happens.
Creating New MCP Server: Tariff News Reaction
The purpose of this new MCP server is to search the internet for the latest reactions to the recently announced tariffs. It uses DuckDuckGo as a news search engine for simplicity.

Figure. new Tariff News Reaction MCP Server Flow
You can skip this step by download the code here: `https://github.com/minyang-chen/AI-powered-Development/tree/main/tariff-news-server`
To save time, I will use “VS Code + Roo Code + Gemini 2.5 Pro” tool stack to quickly generate the MCP server code and configuration with a single requirement prompt. If you are not familiar with using “VS Code + Roo Code + Gemini 2.5 Pro” for autonomous coding development, please see my previous post here:
AI-powered Autonomous Coding VSCode + Roo Code + Gemini 2.5 Pro
AI-powered Autonomous Coding VSCode + Roo Code + Gemini 2.5 Pro
As large language models (LLMs) become more advanced, tools like Cursor, Dalvin, Cline, Roo Code, and Copilot are…
mychen76.medium.com
Generate Tariff News Reaction MCP Server Code
Enter following Requirement Prompt in Roo Code Chat Window in Code mode.

**Project Goal:**
Create a Python-based MCP server that provides a tool to search for recent news articles concerning international reactions to the US tariffs announced in April 2025, supporting both stdio and SSE transports.

**1. Business Requirements:**

*   The server should enable users (or AI agents) to query for news articles about how different countries are reacting to the specified US tariffs.
*   The focus should be on retrieving relevant news published within the last week.
*   The server should be packaged for easy distribution and installation using pip.

**2. Technical Requirements:**

*   **MCP Server Implementation:**
   *   Implement using the `mcp` Python SDK (package name: `mcp`).
   *   Support both standard I/O (`stdio`) and Server-Sent Events (`sse`) transport mechanisms, selectable via a `--transport` command-line argument.
   *   Employ the decorator-based API (`@mcp_server.list_tools`, `@mcp_server.call_tool`) provided by `mcp.server.lowlevel.Server`.
   *   Use `click` for command-line argument parsing (`--transport`, `--port`).
   *   Use `starlette` and `uvicorn` to handle the web server component for SSE transport.

*   **Core Tool (`get_tariff_reaction_news`):**
   *   **Functionality:** Searches the internet using the DuckDuckGo search engine (`duckduckgo-search` library) for news articles related to tariff reactions.
   *   **Search Query Construction:**
       *   Base query: "reactions to US tariffs April 2025"
       *   If `country` input is provided: "reactions from [Country Name] to US tariffs April 2025"
       *   Append `additional_keywords` if provided.
   *   **Filtering:** Limit search results to news published within the last week (using `timelimit='w'` with `duckduckgo-search`).
   *   **Ranking:** Default DuckDuckGo relevance/ranking.

*   **Tool Schemas (using Pydantic):**
   *   **Input (`GetTariffReactionNewsInput`):**
       *   `country`: `Optional[str]` - The specific country to focus the search on.
       *   `additional_keywords`: `Optional[str]` - Extra terms to add to the query.
   *   **Internal Output Models:**
       *   `SearchResultItem`: Defines the structure for a single result (`title`, `url`, `snippet`, `source`, `published_date`).
       *   `SearchSuccessOutput`: Contains a `list[SearchResultItem]` on success.
       *   `SearchErrorOutput`: Contains an `error: str` field on failure.
   *   **MCP Tool Return Type:** The `@mcp_server.call_tool` decorated function will return `list[mcp.types.TextContent]`. The `SearchSuccessOutput` or `SearchErrorOutput` models will be serialized into the `text` field of the `TextContent` block. Tool execution errors should be raised as standard Python exceptions (e.g., `ValueError`, `Exception`), which the `mcp` library will format into MCP error responses.

*   **Dependencies:**
   *   `mcp[cli]>=1.6.0`: For MCP server/tool implementation and types.
   *   `duckduckgo-search>=2025.4.1`: For executing the web search.
   *   `pydantic>=2.11`: For defining input/output schemas and validation.
   *   `anyio>=4.0`: Required by the `mcp` library for running the asynchronous server via stdio.
   *   `click>=8.0`: For command-line argument parsing.
   *   `starlette>=0.27`: For SSE transport web framework.
   *   `uvicorn[standard]>=0.23`: For running the Starlette application.
   *   `requests>=2.25`: (Included but not directly used).

*   **Development Environment:**
   *   Use a Python virtual environment (e.g., `venv`) to manage dependencies.
   *   Include a `requirements.txt` file (generated via `pip freeze`).

*   **Packaging:**
   *   Use a `pyproject.toml` file with a `setuptools` build backend to make the server installable via `pip`.
   *   Include a console script entry point (`tariff-news-server`) that runs the main CLI function (`main_cli`).

*   **Code Structure:**
   *   Organize code logically (e.g., `schemas.py`, `tool.py`, `server.py` within a `src/tariff_news_server` package).

*   **Execution:**
   *   **Stdio (Default):** Run via `python -m tariff_news_server.server` or `tariff-news-server` (if installed). Configure in MCP settings using the `python -m ...` command.
   *   **SSE:** Run via `tariff-news-server --transport sse [--port <port>]`. Configure in MCP settings using the console script command and arguments, or as a remote SSE server pointing to the correct URL (e.g., `http://localhost:8000/mcp/sse`).

**3. Useful Resources (Original Links):**

*   Python MCP SDK (`mcp` package): [https://github.com/modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk)
*   Example MCP Servers: [https://github.com/modelcontextprotocol/python-sdk/tree/main/examples/servers](https://github.com/modelcontextprotocol/python-sdk/tree/main/examples/servers) (Specifically, `simple-tool` provides a relevant pattern).

**4. Documentation:**
- generate a soluton diagram for tariff news MCP server
- generate documentation for source code
Enter the requirement prompt on VS Code — Roo Code extension chat window.

figure. Enter requirement prompt
“Upon successful completion, a new tariff News Reaction MCP server working codes get generated and updated configuration will be generated for testing in Roo Code.”
Test MCP Server in Roo Code
We can verify Roo Code has Tariff News MCP server enabled in MCP server tab.

figure. Verify Tariff Tool is enabled
Test tool call is working by ask question: “mexico latest reaction on tariff?”

See Final Response.

figure. Test Tools Call
Based on recent news articles retrieved using the tariff-news-server, Mexico was spared from the latest round of US tariffs announced in April 2025.

Here's a summary of the reaction:

Relief and Celebration: Mexico celebrated dodging these new tariffs, which targeted many other US trading partners.
Continued Free Trade: This exclusion suggests that the existing free trade relationship between the US and Mexico might survive this round of tariff implementations.
Awareness of Broader Impact: Despite being spared directly, Mexico acknowledges that the global economic uncertainty created by these widespread tariffs could still negatively affect its economy.
In essence, Mexico is relieved to have avoided the direct impact of the tariffs but remains cautious about the indirect consequences stemming from global trade instability.
So far, looks good.
Add new MCP server to MCPO
Step-1. Install the new MCP server to MCPO server python environment using this command:
$ git clone https://github.com/minyang-chen/AI-powered-Development.git
$ cd tariff-news-server
$ pip install -e .
Step-2. Update MCP settings.json by adding Tariff News Reaction MCP Server
{
 "mcpServers": {
   "memory": {
     "command": "npx",
     "args": ["-y", "@modelcontextprotocol/server-memory"]
   },
   "time": {
     "command": "uvx",
     "args": ["mcp-server-time", "--local-timezone=America/New_York"]
   },
 "fetch": {
   "command": "uvx",
   "args": ["mcp-server-fetch"]
 },
 "tariff": {
   "command": "python3",
   "args": [
     "-m",
     "tariff_news_server.server"
   ]
 }
 }
}
Step-3. Start MCP server
$ uvx mcpo --config ./config.json --port 8001
Step-4. Check MCP Server API Docs is up.

Figure. Tariff News MCP Server Doc
Okay, we completed the MCPO server update now. next move to update Open-WebUI.
Add Tariff News MCP server to Open-WebUI
Step-1. Go to Settings / Tools then click + button

Figure. Add Tariff News MCP Server as Tool
Step 2. Check if the Tariff News Reaction server is enabled on Chat Window

Figure. Verify Tariff News MCP Server
Step 3. To test the tool, try asking a question. For example, you could ask: ‘What is Canada’s latest reaction to tariffs?

Figure. Test Tariff News MCP Server
The test results were successful, resulting in a positive response and initiating a tool call. For more details, please click the link: ‘tool_endpoint_get_tariff_reaction_news_post’. You can also verify the MCPO server tool call in the log file.
## MCPO server Log

INFO:     127.0.0.1:33694 - "OPTIONS /tariff/get_tariff_reaction_news HTTP/1.1" 200 OK
Calling get_tariff_reaction_news with arguments: {'country': 'Canada'}
2025-04-03 20:06:39,935 - mcp.server.lowlevel.server - INFO - Processing request of type CallToolRequest
2025-04-03 20:06:39,935 - __main__ - INFO - Received call_tool request for tool: get_tariff_reaction_news
2025-04-03 20:06:39,935 - __main__ - INFO - Parsed tool input: country='Canada' additional_keywords=None
2025-04-03 20:06:39,935 - tariff_news_server.tool - INFO - Executing search with query: 'reactions from Canada to US tariffs'
2025-04-03 20:06:40,189 - primp - INFO - response: https://duckduckgo.com/?q=reactions+from+Canada+to+US+tariffs 200
2025-04-03 20:06:41,407 - primp - INFO - response: https://duckduckgo.com/news.js?l=wt-wt&o=json&noamp=1&q=reactions+from+Canada+to+US+tariffs&vqd=4-290943568794945560942460956578934889745&p=-2&df=w 200
2025-04-03 20:06:41,408 - tariff_news_server.tool - INFO - Found 10 results.
2025-04-03 20:06:41,408 - __main__ - INFO - Tool execution result type: <class 'tariff_news_server.schemas.SearchSuccessOutput'>
2025-04-03 20:06:41,408 - __main__ - INFO - Tool succeeded, returning 10 results.
INFO:     127.0.0.1:33694 - "POST /tariff/get_tariff_reaction_news HTTP/1.1" 200 OK
Final Thoughts
I like mcpo so far. specially, give you ability to test the MCP server out-of-box using API doc. It’s a simple proxy that allows you to use MCP server commands with tools expecting standard OpenAPI servers. It’s designed to make integration with LLM agents and applications effortless. Mcpo is a valuable tool, especially for existing Open-WebUI users.
Thanks for reading!
I hope you find this article useful.
Have a good day!

