# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

Parent repo containing two **git submodules** from the Orpheam organization:

- **`orpheam-libs/`** — Shared Python framework (v3.0) for building multi-agent LLM apps. Published as the `orpheam` package on a private PyPI (Nexus).
- **`ia/`** — Backend: Celery-based workers that consume RabbitMQ jobs (KG building, embeddings, voice STT/TTS). Communicates with a Laravel backend via RabbitMQ.
  - `ia/workers/` — Worker code and docker-compose
  - `ia/jupyter/` — JupyterLab container for experimentation

**Workflow rule:** Prefer modifying `orpheam-libs/` (the framework) over `ia/workers/` (production code), unless the change is specific to the worker domain. Tests and documentation are written as Jupyter notebooks.

## Development Commands

### orpheam-libs
```bash
cd orpheam-libs
poetry install                           # install deps
pytest tests/                            # run all tests
pytest tests/test_file.py::test_name -v  # single test
```

### ia/workers
```bash
cd ia/workers
poetry install                                    # install deps
pytest tests/ -v --cov=src --cov-report=term-missing  # all tests with coverage
pytest tests/test_file.py::TestClass::test_name -v    # single test
black --check src/ tests/                          # format check
ruff check src/ tests/                             # lint
mypy src/                                          # type check
```

### Docker (workers)

**Two docker-compose files** in `ia/workers/`:

- `docker-compose.yml` — **Production**. Requires the external `orpheam-network` Docker network (created by the main Orpheam repo which runs Laravel, Neo4j, RabbitMQ, Postgres, etc.). Do NOT use in isolation.
- `docker-compose.test.yml` — **Dev/test standalone**. Self-contained infra (Neo4j, LiteLLM) with no external network dependency. Use this for local development and notebooks.

```bash
# Dev / test (standalone, no external infra needed)
cd ia/workers
docker compose -f docker-compose.test.yml up -d          # Neo4j + LiteLLM
docker compose -f docker-compose.test.yml up litellm -d  # LiteLLM only

# Production (requires orpheam-network + full infra running)
docker compose up kg-worker embedding-worker rabbitmq-consumer
docker compose up voice-worker
```

## Architecture

### orpheam-libs — Core Framework

All public methods return `Result[T]` (defined in `core/result.py`) instead of raising exceptions. This is a hard rule for any new code in this package.

Key modules: `agent/` (LangGraph state machine with fluent API), `agent/orchestration/` (local supervisor + distributed A2A with circuit breaker), `memory/` (Redis checkpointing with schema migrations), `tools/` (MCP tool registry with interceptor pipeline), `graph/` (Neo4j GraphRAG client), `llm/` (LiteLLM router with fallback chain), `testing/` (full mock harness).

Patterns: lazy initialization for Redis/Neo4j connections, lifecycle hooks on agents, ordered MCP interceptors, frozen/immutable Result and OrpheamError objects.

Full API docs (French) in `orpheam-libs/DOCUMENTATION.md`.

### ia/workers — KG Pipeline & Voice Assistant

Domain-driven layout under `src/`:
- **`core/`** — Config (pydantic-settings), exceptions, types
- **`domain/kg_builder/`** — Knowledge Graph extraction pipeline
  - `pipeline/` — v2.1 stage-based pipeline (parsing → chunking → embedding → NER → extraction → transformation → enrichment → validation → storage)
  - `pipeline_v3/` — LangGraph-orchestrated pipeline with forced JSON mode, modular nodes, and intelligent caching
  - `agents/extraction/strategies/` — Strategy pattern for extraction modes: GUIDED, OPEN, HYBRID (default)
  - `services/orchestration/` — Pipeline orchestrators including multi-file batch processing with cross-file deduplication
  - `parsers/` — CSV, JSON, PDF, TXT parsers
  - `infrastructure/` — Neo4j connection management
- **`domain/assistant/`** — Jarvis voice assistant (LangGraph orchestrator, STT, TTS)

Celery queues: `kg-pipeline`, `embeddings`, `voice`. Broker: RabbitMQ. Result backend: RPC.

LLM calls go through OpenRouter (default model: `anthropic/claude-3.5-sonnet`).

## Code Style (ia/workers)

- **Black** formatter, line length 100
- **Ruff** linter (pycodestyle, pyflakes, isort, flake8-comprehensions, flake8-bugbear; ignores E501, B008, C901)
- Python 3.10+ target
- Logging via `loguru`

## Language

Code comments, docstrings, and documentation are primarily in **French**.
