# Project Brief: Self-hosted AI Package

## Core Requirements and Goals

The primary goal of the Self-hosted AI Package is to provide an open, Docker Compose-based template for quickly bootstrapping a fully featured local AI and low-code development environment. This environment should include:

*   A platform for running local LLMs (Ollama).
*   A user interface for interacting with local models and AI agents (Open WebUI).
*   A robust database, vector store, and authentication solution (Supabase).
*   A low-code platform for building workflows and integrating services (n8n).
*   A no/low-code AI agent builder (Flowise).
*   A high-performance vector store (Qdrant).
*   An open-source metasearch engine for privacy-respecting web search (SearXNG).
*   A service for managed HTTPS/TLS (Caddy).
*   An LLM engineering platform for agent observability (Langfuse).

The package aims to be easy to install and use locally via Docker Compose, with support for different GPU configurations (Nvidia, AMD, CPU-only, or connecting to a native Ollama instance). It should also provide guidance for deployment to cloud environments.

## Key Features

*   Pre-configured Docker Compose setup for included services.
*   Automated startup script (`start_services.py`) to manage service dependencies and profiles.
*   Support for different Ollama configurations (CPU, GPU, external).
*   Pre-loaded n8n workflows for local RAG AI agents.
*   Integration instructions for Open WebUI and n8n.
*   Guidance on environment variable setup and security.
*   Troubleshooting tips for common issues.
*   Instructions for upgrading services.

## Target Audience

Developers and users who want to experiment with or build self-hosted AI applications and workflows using a pre-packaged set of popular open-source tools.

## Scope

The project focuses on providing the core infrastructure and initial configuration to get the included services running and integrated. It includes basic examples (like the pre-loaded n8n workflows) but does not aim to be a comprehensive application platform itself. Custom application development on top of this stack is outside the direct scope of this package, though the package provides the foundation for such development.

## Constraints

*   Relies on Docker and Docker Compose for deployment.
*   Requires manual environment variable setup for secrets.
*   Cloud deployment guidance is basic and requires user expertise.
*   GPU support is dependent on host system configuration and Docker setup.
