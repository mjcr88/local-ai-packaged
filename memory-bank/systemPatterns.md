# System Patterns and Architecture

This document outlines the key architectural patterns and technical decisions implemented in the Self-hosted AI Package, focusing on the Docker Compose setup and service orchestration.

## Docker Compose Structure and Orchestration

The project utilizes Docker Compose to manage multiple services. The core setup involves two main Docker Compose files:
1.  `supabase/docker/docker-compose.yml`: Defines the Supabase services.
2.  `docker-compose.yml` (at the root): Defines the main application stack services (n8n, Ollama, Open WebUI, Flowise, Qdrant, Caddy, Langfuse, SearXNG, Redis).

The `start_services.py` script is the primary tool for managing the lifecycle of these services. It orchestrates a two-stage startup process:
1.  **Supabase First:** The script explicitly starts the services defined in `supabase/docker/docker-compose.yml` using `docker compose -p localai -f supabase/docker/docker-compose.yml up -d`.
2.  **Initialization Pause:** A 10-second pause (`time.sleep(10)`) is included to allow Supabase services to initialize before the main stack is brought up.
3.  **Main Stack Second:** The script then starts the services defined in the root `docker-compose.yml` using `docker compose -p localai -f docker-compose.yml up -d`, potentially including a `--profile` flag based on user input.

All services, regardless of which compose file they are defined in, are grouped under the unified Docker Compose project name `localai` using the `-p localai` flag. This allows them to be managed together (e.g., viewed in Docker Desktop).

## Docker Compose Profile Handling

The `start_services.py` script accepts a `--profile` argument which influences which services are started from the root `docker-compose.yml`:

*   `--profile cpu`: Activates the `cpu` profile. This starts the `ollama-cpu` service (which uses the standard `ollama/ollama:latest` image) and its associated `ollama-pull-llama-cpu` initialization service.
*   `--profile gpu-nvidia`: Activates the `gpu-nvidia` profile. This starts the `ollama-gpu` service (configured for Nvidia GPUs) and its associated `ollama-pull-llama-gpu` initialization service.
*   `--profile gpu-amd`: Activates the `gpu-amd` profile. This starts the `ollama-gpu-amd` service (configured for AMD GPUs using the `ollama/ollama:rocm` image) and its associated `ollama-pull-llama-gpu-amd` initialization service.
*   `--profile none`: This is a special case handled by `start_services.py`. When this profile is selected, the script executes `docker compose -p localai -f docker-compose.yml up -d` *without* specifying any `--profile` flag. In Docker Compose, services defined with a `profiles:` key are *only* started if one of their profiles is explicitly activated. Therefore, using `--profile none` ensures that the Dockerized Ollama services (which all have `profiles:` keys) are **not** started. This is intended for users who are running Ollama natively on their host machine or elsewhere, allowing the other services (which typically do not have `profiles:` keys) to start and connect to the external Ollama instance.

Services defined in the root `docker-compose.yml` that **do not** have a `profiles:` key are considered "default" services and will be started by `start_services.py` regardless of the `--profile` argument used (including `--profile none`).

## Environment Variable Management

Configuration and secrets are primarily managed via the root `.env` file.
*   For Supabase services (defined in `supabase/docker/docker-compose.yml`), the `start_services.py` script explicitly copies the root `.env` file to `supabase/docker/.env` before starting Supabase.
*   For services in the root `docker-compose.yml`, Docker Compose automatically reads the root `.env` file and makes those variables available to service containers where they are referenced (e.g., `environment: - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}`).

## SearXNG Specific Initialization

The `start_services.py` script includes specific logic to handle the initial setup requirements for the SearXNG service:
1.  **Secret Key Generation:** The `generate_searxng_secret_key()` function checks for `searxng/settings.yml`. If it doesn't exist, it copies `searxng/settings-base.yml`. It then uses `openssl` and `sed` (or PowerShell on Windows) to replace the `ultrasecretkey` placeholder in `searxng/settings.yml` with a randomly generated hexadecimal string.
2.  **First-Run `cap_drop` Adjustment:** The `check_and_fix_docker_compose_for_searxng()` function detects if SearXNG is running for the first time (by checking for the existence of `uwsgi.ini` inside the container). If it is the first run, it temporarily comments out the `cap_drop: - ALL` line in the root `docker-compose.yml` for the `searxng` service. This grants SearXNG broader permissions needed for its initial setup. On subsequent runs, it detects that initialization is complete and re-enables (uncomments) the `cap_drop: - ALL` line to restore the stricter security configuration.

These automated steps simplify the initial setup process for SearXNG, which otherwise might require manual file edits or troubleshooting related to container permissions.
