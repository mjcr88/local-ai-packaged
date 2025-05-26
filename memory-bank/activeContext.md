# Active Context: Self-hosted AI Package

## Current Work Focus

The current focus is on initializing the memory bank to cover the entire codebase, including the README and project summary, and to incorporate more detailed information such as port levels for services, container naming conventions, and details outlined in any of the files found in the codebase.

## Recent Changes

*   Created `memory-bank/projectbrief.md` based on the README and project goals.
*   Created `memory-bank/productContext.md` based on the project's purpose, problems solved, and user experience goals.
*   Updated `memory-bank/techContext.md` to include detailed port URLs for all services and other important technical details gathered from `docker-compose.yml` and `supabase/docker/docker-compose.yml`.
*   Updated `memory-bank/systemPatterns.md` based on the analysis of `README.md`, `docker-compose.yml`, and `start_services.py`, focusing on Docker Compose structure, orchestration, profile handling, environment variables, and SearXNG initialization.
*   Created `memory-bank/progress.md` to document the current status, what works, what's left, and known issues.

## Next Steps

1.  Review all created memory bank files to ensure they accurately reflect the project based on the available documentation and newly gathered details.
2.  Confirm the memory bank initialization is complete.

## Active Decisions and Considerations

*   The memory bank structure follows the defined hierarchy: `projectbrief` -> (`productContext`, `systemPatterns`, `techContext`) -> `activeContext` -> `progress`.
*   Information is being extracted from `README.md`, `project_summary.md`, `docker-compose.yml`, and `supabase/docker/docker-compose.yml` to populate the memory bank files.
*   The `systemPatterns.md` file specifically details the Docker Compose and `start_services.py` orchestration logic.
*   The `techContext.md` now includes explicit port mappings for all services.
*   **Adding New Services:** To add a new Docker service to this stack:
    1.  Define the new service within the `services:` section of the root `docker-compose.yml`.
    2.  If the service should run regardless of the selected Ollama profile (i.e., always on, including with `--profile none`), **do not** include a `profiles:` key in its definition.
    3.  If the service is specific to a particular Ollama setup, assign it to the relevant profile (e.g., `profiles: ["cpu"]`).
    4.  Add any necessary environment variables for the new service to the root `.env` file and reference them in the service's `environment:` section in `docker-compose.yml`.
    5.  Run `python start_services.py --profile <your-profile>` to start the services, including your new one.

## Learnings and Project Insights

*   The project uses a two-stage startup process managed by a Python script.
*   Docker Compose profiles are used to conditionally start different Ollama service configurations.
*   SearXNG has specific first-run initialization steps handled by the startup script.
*   The `.env` file is central to configuration, with a specific step in the script to copy it for the Supabase services.
*   Detailed port mappings for all services have been identified and documented.
