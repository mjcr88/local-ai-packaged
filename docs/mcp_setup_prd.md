## Product Requirements Document: MCP Server Integration via `mcpo`

__1. Introduction & Goal__

- __Goal:__ To integrate 'get time' and 'Ardoq' MCP servers into the existing `local-ai-packaged` environment. This will be achieved by deploying `mcpo` (MCP OpenAPI Proxy) in a new, standalone Docker Compose stack. This `mcpo` stack will communicate with the existing Open WebUI and n8n services (and potentially future services) via a shared Docker network.
- __Purpose:__ Enable Open WebUI and n8n to utilize the functionalities of the 'time' and Ardoq MCP servers through a standard OpenAPI interface provided by `mcpo`. This approach avoids complex modifications to the existing Docker Compose stack and promotes modularity.

__2. Prerequisites__

- [ ] Docker and Docker Compose installed and operational.
- [ ] The `local-ai-packaged` repository cloned and functional.
- [ ] The Ardoq MCP server code downloaded locally.
- [ ] Ardoq API Token and Ardoq API URL (to be provided by the user during setup).
- [ ] Node.js and npm installed on the host machine (for installing Ardoq MCP server dependencies).
- [ ] `uv` installed on the host machine (if not already, `mcpo` uses `uvx`).

__3. Proposed Directory Structure__

To maintain a clean repository root, MCP server configurations and related files will be organized as follows:

```javascript
/Users/mj/Developer/local-ai-packaged/
├── docker-compose.yml               # Existing main Docker Compose file
├── mcpo-docker-compose.yml          # New Docker Compose file for mcpo
├── mcpo-config.json                 # Configuration for mcpo listing MCP servers
├── mcp-servers/                     # New directory for custom MCP server code
│   └── ardoq-mcp/                   # User will place Ardoq server files here
│       ├── index.js
│       ├── src/
│       ├── package.json
│       └── ... (other Ardoq server files)
├── .env                             # Existing environment file (will be updated)
└── ... (other existing files and directories)
```

__4. Detailed Plan__

__4.1. Network Configuration__

- [ ] A new Docker network named `local_ai_network` will be created.
- [ ] This network will be defined in the existing `docker-compose.yml` to ensure it's managed as part of the primary stack.
- [ ] The new `mcpo-docker-compose.yml` will reference this network as `external`.
- [ ] The `open-webui` and `n8n` services in the existing `docker-compose.yml` will be updated to connect to `local_ai_network` in addition to their default network.

__4.2. `mcpo` Configuration (`mcpo-config.json`)__

- [ ] This JSON file will instruct `mcpo` which MCP servers to proxy.

- [ ] It will define:

  - [ ] The 'time' server, launched using `uvx mcp-server-time`.
  - [ ] The 'ardoq' server, launched using `node /app/servers/ardoq-mcp/index.js`.
  - [ ] Environment variables (`ARDOQ_API_TOKEN`, `ARDOQ_API_URL`) will be passed to the Ardoq server process. These will be sourced from the `mcpo` container's environment.

__4.3. New `mcpo-docker-compose.yml`__

- [ ] This file will define the `mcpo` service.

- [ ] __Image:__ `ghcr.io/open-webui/mcpo:main`.

- [ ] __Ports:__ Expose host port `8001` mapped to container port `8000` (port `8001` is confirmed available).

- [ ] __Volumes:__

  - [ ] Mount `mcpo-config.json` into the container.
  - [ ] Mount the user-provided Ardoq MCP server directory (e.g., `./mcp-servers/ardoq-mcp`) into the container at `/app/servers/ardoq-mcp`.

- [ ] __Environment Variables:__
  - [ ] Pass `ARDOQ_API_TOKEN` and `ARDOQ_API_URL` from the host's environment (read from the root `.env` file) to the `mcpo` container.

- [ ] __Command:__ Instruct `mcpo` to use the mounted `mcpo-config.json` and listen on port `8000`.

- [ ] __Networks:__ Connect the `mcpo` service to the `local_ai_network`.

- [ ] __Restart Policy:__ `always` for automatic startup.

__4.4. Modifications to Existing `docker-compose.yml`__

- [ ] Add the definition for the `local_ai_network`.
- [ ] Modify the `open-webui` and `n8n` service definitions to include `local_ai_network` in their `networks` list.

__4.5. Environment Variables (`.env` file)__

- [ ] The user will need to add `ARDOQ_API_TOKEN` and `ARDOQ_API_URL` to their root `.env` file. These values will be read by `mcpo-docker-compose.yml` and passed to the `mcpo` container.

__5. Step-by-Step Implementation Guide__

__Step 1: Prepare Ardoq MCP Server Files__ * __Action (User):__ - [ ] Create the directory `mcp-servers/ardoq-mcp` in the root of the `local-ai-packaged` repository. - [ ] Copy all files and subdirectories from your local Ardoq MCP server repository into `./mcp-servers/ardoq-mcp/`. - [ ] Navigate to the `./mcp-servers/ardoq-mcp/` directory in your terminal. - [ ] Run `npm install` to install its Node.js dependencies. This creates the `node_modules` directory within `./mcp-servers/ardoq-mcp/`. * __Test Checkpoint:__ - [ ] Verify that `npm install` completes without errors. - [ ] Verify the `node_modules` directory exists in `./mcp-servers/ardoq-mcp/`.

__Step 2: Update Root `.env` File__ * __Action (User):__ - [ ] Open the `.env` file in the root of the `local-ai-packaged` repository. - [ ] Add the following lines, replacing the placeholder values with your actual Ardoq credentials: `env ARDOQ_API_TOKEN="YOUR_ARDOQ_API_TOKEN_HERE" ARDOQ_API_URL="https://your-instance.ardoq.com" `* __Test Checkpoint:__ - [ ] Ensure the `.env` file is saved with the correct Ardoq credentials.

__Step 3: Create `mcpo-config.json`__ * __Action (Cline):__ Create a file named `mcpo-config.json` in the root of the `local-ai-packaged` repository with the following content: `json { "mcpServers": { "time": { "command": "uvx", "args": ["mcp-server-time"] }, "ardoq": { "command": "node", "args": ["/app/servers/ardoq-mcp/index.js"], "env": { "ARDOQ_API_TOKEN": "${ARDOQ_API_TOKEN}", "ARDOQ_API_URL": "${ARDOQ_API_URL}" } } } } `* __Test Checkpoint:__ - [ ] Verify the file `mcpo-config.json` is created with the correct content.

__Step 4: Create `mcpo-docker-compose.yml`__ * __Action (Cline):__ Create a file named `mcpo-docker-compose.yml` in the root of the `local-ai-packaged` repository with the following content: ```yaml version: '3.8'

````javascript
    services:
      mcpo:
        image: ghcr.io/open-webui/mcpo:main
        container_name: mcpo
        restart: always
        ports:
          - "8001:8000"
        volumes:
          - ./mcpo-config.json:/app/config.json:ro
          # This path assumes the Ardoq server is in ./mcp-servers/ardoq-mcp/ relative to this compose file
          - ./mcp-servers/ardoq-mcp:/app/servers/ardoq-mcp:ro
        environment:
          - ARDOQ_API_TOKEN=${ARDOQ_API_TOKEN}
          - ARDOQ_API_URL=${ARDOQ_API_URL}
        command: ["--config", "/app/config.json", "--port", "8000"]
        networks:
          - local_ai_network

    networks:
      local_ai_network:
        external: true
        name: local_ai_network # Explicitly naming the external network
    ```
*   **Test Checkpoint:**
    - [ ] Verify the file `mcpo-docker-compose.yml` is created with the correct content.
````

__Step 5: Modify Existing `docker-compose.yml`__ * __Action (Cline):__ Modify the existing `docker-compose.yml` file. 1. Add the `local_ai_network` definition at the bottom of the file. 2. Add `local_ai_network` to the `networks` section of the `open-webui` and `n8n` services. * __Test Checkpoint:__ - [ ] Verify `docker-compose.yml` is updated correctly with the new network definition and service connections.

__Step 6: Start the Services__ * __Action (User):__ - [ ] Ensure no existing Docker containers are using port `8001`. - [ ] First, bring up the main stack (which now defines `local_ai_network`). You can use your `start_services.py` script or run: `bash docker compose -p localai -f docker-compose.yml up -d `- [ ] Then, start the `mcpo` stack: `bash docker compose -f mcpo-docker-compose.yml up -d `* __Test Checkpoint:__ - [ ] Run `docker network ls` and verify `local_ai_network` exists. - [ ] Run `docker ps` and verify `mcpo`, `open-webui`, and `n8n` containers are running. - [ ] Check `mcpo` container logs: `docker logs mcpo`. Look for messages indicating it has started and is proxying the 'time' and 'ardoq' servers.

__6. Testing and Verification__

- __Test 1: `mcpo` API Documentation__

  - [ ] Open a web browser and navigate to:

    - [ ] `http://localhost:8001/time/docs`
    - [ ] `http://localhost:8001/ardoq/docs`

  * __Expected Result:__ Interactive OpenAPI (Swagger UI) documentation should be displayed for both the 'time' and 'ardoq' MCP servers, listing their available tools.

- __Test 2: 'time' Server via `mcpo`__

  - [ ] Use `curl` or Postman to call a 'time' server tool, e.g., `get_current_time_utc`.

    ```bash
    curl -X POST http://localhost:8001/time/get_current_time_utc -H "Content-Type: application/json" -d '{}'
    ```

  * __Expected Result:__ A JSON response containing the current UTC time.

- __Test 3: 'ardoq' Server `verify-connection` via `mcpo`__

  - [ ] Use `curl` or Postman to call the `verify-connection` tool from the Ardoq server.

    ```bash
    curl -X POST http://localhost:8001/ardoq/verify-connection -H "Content-Type: application/json" -d '{}'
    ```

  * __Expected Result:__ A JSON response indicating successful connection to Ardoq (or an error if credentials are incorrect).

- __Test 4: Open WebUI Integration__

  - [ ] Open Open WebUI (e.g., `http://localhost:3000`).
  - [ ] Go to Settings > Tools > Add Tool.
  - [ ] Enter the URL for the `mcpo` service: `http://localhost:8001`.
  - [ ] Save the tool configuration.
  - [ ] Attempt to use a tool from the 'time' or 'ardoq' server in a chat.

  * __Expected Result:__ Open WebUI should list the tools from `mcpo`, and tool calls should execute successfully, with results appearing in the chat.

- __Test 5: n8n Integration__

  - [ ] Open n8n (e.g., `http://localhost:5678`).

  - [ ] Create a new workflow.

  - [ ] Use an "HTTP Request" node to call a tool from the 'time' or 'ardoq' server via `mcpo`.

    - URL: `http://mcpo:8000/time/get_current_time_utc` or `http://mcpo:8000/ardoq/verify-connection` (using service name as n8n is on the same Docker network).
    - Method: POST
    - Body: `{}` (empty JSON object if no arguments)
    - Headers: `Content-Type: application/json`

  - [ ] Execute the workflow.

  * __Expected Result:__ The HTTP Request node should execute successfully and return the tool's output.

__7. Rollback Plan (Optional but Recommended)__

- [ ] Stop and remove the `mcpo` Docker Compose stack: `docker compose -f mcpo-docker-compose.yml down --volumes`
- [ ] Revert changes made to the existing `docker-compose.yml` (remove network definitions for `open-webui`, `n8n`, and the `local_ai_network` definition).
- [ ] Remove `mcpo-config.json` and `mcpo-docker-compose.yml`.
- [ ] Remove Ardoq credentials from the `.env` file.
- [ ] Restart the main Docker Compose stack: `docker compose -p localai -f docker-compose.yml up -d`
