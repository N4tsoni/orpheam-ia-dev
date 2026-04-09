# Orpheam Libs — Le Framework

## C'est quoi ?

`orpheam-libs` est le framework Python partagé pour construire les agents IA. Publié comme package `orpheam`. Tout le code réutilisable vit ici — les workers ne contiennent que du glue code.

## Modules

| Module | Classe principale | En bref |
|--------|-------------------|---------|
| `core/` | `Result[T]`, `OrpheamError` | Monade erreur — pas d'exceptions dans l'API publique |
| `agent/` | `OrpheamStateAgent` | Wrapper LangGraph avec API fluent (`.add_step().build()`) |
| `agent/orchestration/` | `OrpheamSupervisor`, `OrpheamA2ASupervisor` | Multi-agent local + distribué (HTTP, circuit breaker) |
| `tools/` | `OrpheamToolRegistry` | Registre MCP avec intercepteurs ordonnés |
| `llm/` | `OrpheamRouter` | Gateway LiteLLM avec fallback chain |
| `memory/` | `OrpheamRedisMemory` | Checkpointing Redis + migrations de schéma |
| `graph/` | `OrpheamGraphClient`, `OrpheamGraphitiClient` | Neo4j, Graphiti, parsers, pipeline KG |
| `testing/` | `OrpheamTestHarness` | Mocks complets pour tester sans infra |

## Patterns

- **Result[T]** — toutes les méthodes publiques retournent `Result`, jamais d'exception
- **Lazy init** — Redis/Neo4j se connectent au premier appel
- **Fluent Builder** — chainage `.add_step().set_entry().build()`
- **Circuit Breaker** — sur l'orchestration distribuée A2A

## Dépendances

Core : `pydantic v2`, `litellm`, `httpx`. Le reste est optionnel via extras (`redis`, `neo4j`, `langgraph`, `mcp`...).

Python >= 3.10.
