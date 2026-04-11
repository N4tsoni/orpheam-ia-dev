# TODO Phase 1 — Promethee : Pipeline KG

> Objectif : pipeline fiable et mesuré pour construire des Knowledge Graphs à partir de documents.
> **Statut** : en finalisation ✅

---

## 1.1 Intégration LiteLLM ✅
- [x] Service `litellm` dans `docker-compose.yml` (container, healthcheck, port 4000)
- [x] `litellm_config.yaml` — 4 modèles via OpenRouter (Gemini Flash, Gemini Lite, Embeddings, Claude)
- [x] `OrpheamRouter` utilise `api_base` = gateway LiteLLM
- [x] `OrpheamGraphitiClient` refactoré — provider-agnostique via LiteLLM
- [x] Proxy testé en standalone (notebook 01)

## 1.2 Pipeline KG ✅
- [x] Rework parsers → orpheam-libs (BaseParser, ParserRegistry, 6 formats)
- [x] Pipeline orchestration → orpheam-libs (LangGraph steps, BatchOrchestrator)
- [x] Audit Graphiti vs étapes custom — Graphiti gère ~85% nativement
- [x] Multi-format : CSV, PDF, TXT, JSON, XLSX, DOCX + registry auto-detect
- [x] 33 tests parsers passent
- [x] KGPipelineConfig — objet centralisé pour tous les paramètres tunables

## 1.3 Pipeline V3 Neuro-Symbolique ✅ (Laz)
- [x] Chemin classique (CSV/JSON) — 4 nœuds : Entity → Relation → Validation → Storage
- [x] Chemin neuro-symbolique (Markdown) — 7 nœuds : Markdown → SCT → Entity → Relation → ASP → Validation → Storage
- [x] SCT (GNN + Adaptive Fusion), ASP (clingo + réparation auto)
- [x] Forced JSON mode → 100% de parsing réussi
- [ ] Tests d'intégration pipeline V3 dans la nouvelle archi
- [ ] Notebook dédié pipeline V3

## 1.4 Parser Worker (file2md) ✅
- [x] Parsing hybride PDF — MarkItDown (trivial) + Marker (complexe) par page
- [x] Infra MinIO — S3 client, docker-compose, config
- [x] Celery parse-worker — task `process_parsing`, queue `parsing`
- [x] Branchement Laravel → parse-worker (DTOs, consumer dispatch)
- [x] Chaînage parse-worker → Pipeline V3 (flag `chain_to_kg`)
- [x] Choix du parser configurable : `auto`, `marker`, `markitdown`, `sanitize`
- [x] Docker compose test — RabbitMQ + parse-worker standalone
- [x] Makefile — `up-worker`, `up-infra`, `build-worker`, `logs-worker`, `status`
- [ ] Build + test parse-worker Docker
- [ ] Tests E2E — notebook 07 complet
- [ ] Côté Laravel — `Storage::disk('s3')` + job RabbitMQ queue `parsing`

## 1.5 Benchmark & Évaluation
- [x] Guide d'évaluation — `docs/Evaluation Pipeline KG - Guide.md`
- [ ] Gold standard — 10-15 documents annotés (entités + relations attendues)
- [ ] Métriques extraction : Precision, Recall, F1 (exact + fuzzy + sémantique)
- [ ] Métriques retrieval : Hit Rate, MRR, Precision@K, RAGAS
- [ ] Benchmark A/B — comparer modèles LLM, modes de recherche, embeddings
- [ ] Notebook benchmark avec tableau comparatif + export CSV

## 1.6 Optimisation Pipeline
- [ ] Cache LLM Redis via LiteLLM
- [ ] Async ingestion — paralléliser `add_episode()`
- [ ] Retry + circuit breaker sur erreurs LLM
- [ ] Token budget — tracking et limites de coût par run
- [ ] Batch embedding
- [ ] Monitoring par étape (Flower/Grafana)

## 1.7 Graph Data Science (Neo4j GDS)
- [ ] Centralité — entités les plus connectées/importantes
- [ ] Communautés — clusters d'entités (Louvain, Leiden)
- [ ] Similarité structurelle — patterns de relations similaires
- [ ] Chemins — connexions entre deux entités
- [ ] Multi-Factor Ranking (centralité + similarité sémantique)
- [ ] Notebook de démonstration GDS

---

## Prochaines étapes immédiates

- [ ] **Build + test parse-worker Docker** — `make build-worker && make up-worker`
- [ ] **Notebook 07 E2E** — test complet parse-worker Docker → MinIO → KG
- [ ] **Mettre à jour notebook 06** — sorties benchmark-ready
- [ ] **Gold standard** — commencer annotation des documents de test
- [ ] **Laravel ↔ MinIO ↔ RabbitMQ** — côté Laravel pour mise en prod
