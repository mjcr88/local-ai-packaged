# Technical Context: Self-hosted AI Package

## Technologies Used

*   **Docker & Docker Compose:** Core technology for containerization and service orchestration.
*   **Python:** Used for the `start_services.py` script to manage service startup and initial configuration.
*   **Ollama:** Platform for running local Large Language Models.
*   **Open WebUI:** Web interface for interacting with LLMs and n8n agents.
*   **Supabase:** Provides PostgreSQL database, PostgREST API, GoTrue (Auth), Realtime, and Storage. Used for database, vector store (pgvector), and authentication.
*   **n8n:** Low-code workflow automation platform with numerous integrations and AI nodes.
*   **Flowise:** No/low-code UI for building AI agents using LangChain.
*   **Qdrant:** Vector database, used as an alternative or supplement to Supabase's pgvector for vector storage.
*   **SearXNG:** Privacy-respecting metasearch engine.
*   **Caddy:** Web server used for managed HTTPS and reverse proxying, particularly for cloud deployments.
*   **Langfuse:** Open-source platform for observing and debugging LLM applications and agents.
*   **Redis/Valkey:** Used by Langfuse for caching and potentially other services.

## Development Setup

*   Requires Docker, Docker Desktop (for Mac/Windows), Git, and Python installed on the host machine.
*   Configuration is managed via a root `.env` file, copied to `supabase/docker/.env` by the startup script.
*   GPU support requires specific Docker configurations and potentially host-level driver setup (Nvidia, AMD).
*   Local access is primarily via `localhost` on various ports:
    *   **n8n:** `http://localhost:5678/`
    *   **Open WebUI:** `http://localhost:3000/`
    *   **Flowise:** `http://localhost:3001/`
    *   **Ollama:** `http://localhost:11434/` (if running in Docker)
    *   **Qdrant:** `http://localhost:6333/`
    *   **Langfuse Web:** `http://localhost:3002/`
    *   **SearXNG:** `http://localhost:8080/`
    *   **Supabase Studio:** `http://localhost:3000/` (via Kong, if configured)
    *   **Supabase Kong (API Gateway):** `http://localhost:8000/` (HTTP), `http://localhost:8443/` (HTTPS)
    *   **Supabase Postgres:** `localhost:5433` (direct access, internal to Langfuse)
    *   **Supabase Pooler:** `localhost:5432` (primary Postgres access for Supabase services)
    *   **Langfuse Clickhouse:** `localhost:8123` (HTTP), `localhost:9000` (native)
    *   **Minio (Langfuse S3):** `http://localhost:9090/` (API), `http://localhost:9091/` (Console)
    *   **Redis/Valkey:** `localhost:6379` (internal to Langfuse)

## Technical Constraints

*   Performance is dependent on host machine resources (CPU, RAM, GPU).
*   Network configuration relies on Docker's internal networking and exposed ports.
*   Cloud deployment requires manual DNS setup and potentially firewall configuration.
*   Secrets must be manually generated and added to the `.env` file.
*   Compatibility with specific GPU drivers and operating systems can be a factor.

## Tool Usage Patterns

*   `start_services.py`: Used for initial setup (SearXNG secret) and orchestrated startup with profile selection.
*   Docker Compose: Used for defining and running the multi-container application.
*   `.env` file: Centralized configuration management.
*   n8n workflows: JSON files (`.json`) defining automation logic.
*   Flowise flows: JSON files (`.json`) defining AI agent structures.
*   Supabase migrations/initialization: Handled via files in `supabase/docker/volumes/db/init/` and potentially `supabase/docker/dev/data.sql`.
*   SearXNG configuration: Managed via `searxng/settings.yml`.
*   Caddy configuration: Defined in `Caddyfile`.
