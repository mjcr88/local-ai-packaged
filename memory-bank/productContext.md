# Product Context: Self-hosted AI Package

## Why This Project Exists

The Self-hosted AI Package addresses the need for developers and users to easily set up a local environment for building and experimenting with AI applications and workflows without relying on external cloud services for core components like LLMs, databases, and vector stores. It provides a pre-configured stack of popular open-source tools that are known to work well together, reducing the complexity of manual setup and integration.

## Problems It Solves

*   **Complex Setup:** Manually installing and configuring multiple interconnected services (LLMs, databases, vector stores, workflow engines, UIs) can be time-consuming and error-prone. This package provides a single `docker-compose` setup.
*   **Privacy Concerns:** Using external AI services or databases may raise privacy concerns. The self-hosted nature keeps data and processing local.
*   **Integration Challenges:** Connecting different tools (like an LLM UI to a workflow engine, or a workflow engine to a database/vector store) requires specific configuration. The package provides pre-configured connections and integration examples.
*   **GPU Utilization:** Setting up GPU access for local LLMs within a containerized environment can be tricky. The `start_services.py` script simplifies this with profiles for different GPU types.
*   **Getting Started:** Provides a quick way for users to start building AI workflows with pre-loaded examples and necessary tools.

## How It Should Work

The package should allow a user to:

1.  Clone the repository.
2.  Set up necessary environment variables (secrets).
3.  Run a single script (`start_services.py`) to launch all required services via Docker Compose, selecting the appropriate profile for their hardware.
4.  Access the various services (n8n, Open WebUI, Supabase, etc.) via their local ports.
5.  Easily integrate the services using provided credentials and instructions (e.g., connecting Open WebUI to n8n, n8n to Ollama/Supabase/Qdrant).
6.  Have pre-configured n8n workflows available to demonstrate RAG agent capabilities.
7.  Optionally configure Caddy for managed HTTPS if deploying to a server with custom domains.
8.  Easily update the services to newer versions.

## User Experience Goals

*   **Simplicity:** The setup process should be as straightforward as possible, primarily involving cloning, configuring `.env`, and running a script.
*   **Speed:** Users should be able to get the environment up and running quickly.
*   **Flexibility:** Support for different hardware configurations (CPU/GPU) and the option to use an external Ollama instance.
*   **Completeness:** Provide a comprehensive set of tools needed for common AI workflow development tasks.
*   **Guidance:** Offer clear documentation for installation, usage, integration, and troubleshooting.
