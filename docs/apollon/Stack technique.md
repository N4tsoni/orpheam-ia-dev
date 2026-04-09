# Apollon — Stack technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| LangGraph | `langgraph ^0.2.0` | Machine a états conversationnelle |
| LangChain | `langchain ^0.3.0` | Abstractions LLM et messages |
| OrpheamRouter | orpheam-libs | Gateway LLM (LiteLLM) |
| OrpheamGraphClient | orpheam-libs | Recherche KG (Neo4j) |
| OrpheamRedisMemory | orpheam-libs | Checkpointing conversations (Redis) |
| Celery | Queue `ml-inference` | Exécution async |

## Tâche Celery

| Task | Queue | Retries |
|------|-------|---------|
| `process_ml_inference` | ml-inference | 2 (30s delay) |

Legacy — sera remplacé par un orchestrateur MCP multi-agent (Sprint 4).

## Fichiers clés

```
ia/workers/orpheam_workers/apollon/
├── tasks.py          # Tâche Celery
├── graph.py          # Factory singleton LangGraph
├── state.py          # AgentState (TypedDict)
├── nodes/            # 11 noeuds LangGraph
├── prompts/          # System prompts par noeud
└── services/
    └── kg_summary_cache.py
```
