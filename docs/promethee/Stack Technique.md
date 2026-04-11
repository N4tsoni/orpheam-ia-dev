# Promethee — Stack Technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| Graphiti (by Zep) | `graphiti-core ^0.3.0` | Extraction KG temporelle |
| Neo4j | `neo4j ^5.15.0` | Base de données graphe |
| LangGraph | `langgraph ^0.2.0` | Orchestration pipeline |
| LiteLLM | Proxy port 4000 | Gateway LLM |
| SentenceTransformers | `all-MiniLM-L6-v2` | Embeddings locaux (384 dims) |
| Marker | `marker-pdf ^1.0.0` | Parsing PDF/DOCX (OCR + vision) |
| MarkItDown | `markitdown ^0.1.0` | Parsing PDF léger (pdfminer) |
| Celery | Queues `kg-pipeline` + `embeddings` + `parsing` | Exécution async |

## Modèles LLM

| Modèle | Rôle |
|--------|------|
| `google/gemini-2.5-flash` | Extraction KG (NER, relations, dedup) |
| `google/gemini-2.0-flash-lite-001` | Tâches légères (reranking) |
| `openai/text-embedding-3-small` | Embeddings via OpenRouter |

## Tâches Celery

| Task | Queue | Description | Retries |
|------|-------|-------------|---------|
| `process_kg_extraction` | kg-pipeline | Extraction d'un document | 3 (60s) |
| `process_kg_batch` | kg-pipeline | Batch multi-fichiers + dedup cross-file | 2 (120s) |
| `process_entity_linking` | kg-pipeline | Liaison Wikidata/DBpedia | 3 |
| `process_kg_graph_rebuild` | kg-pipeline | Reconstruction du graphe | - |
| `process_embedding_generation` | embeddings | Génération vectorielle | 3 |
| `process_parsing` | parsing | Conversion document → Markdown | 2 (30s) |

## Fichiers clés

```
ia/workers/orpheam_workers/promethee/
├── tasks.py              # Tâches Celery KG
├── orchestration.py      # KGOrchestrator + BatchOrchestrator (singletons)
├── neo4j_service.py      # Factory : OrpheamGraphClient + OrpheamGraphitiClient
├── entity_linking.py     # EntityLinker (Wikidata + DBpedia)
└── vector_store.py       # SentenceTransformers → Neo4j vector index

ia/workers/orpheam_workers/parsing/
├── tasks.py              # Tâche Celery parsing
├── pipeline.py           # parse_document() — point d'entrée
├── converter.py          # MarkerPool + MarkItDown
├── complexity.py         # Analyse complexité pages PDF
├── normalize.py          # Enrichissement typographique
├── tables.py             # Extraction tableaux pymupdf
└── sanitize.py           # Nettoyage Markdown

orpheam-libs/orpheam/graph/
├── client.py             # OrpheamGraphClient (Cypher, GraphRAG)
├── graphiti_client.py    # OrpheamGraphitiClient (via LiteLLM)
├── search_config.py      # OrpheamSearchConfig (modes de recherche)
├── parsers/              # CSV, PDF, TXT, JSON, XLSX, DOCX
└── pipeline/             # LangGraph pipeline + BatchOrchestrator
```
