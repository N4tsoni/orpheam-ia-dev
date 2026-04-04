# TODO - Sprint IA

## Sprint 1 — Intégration LiteLLM ✅

### 1.1 Configuration du proxy LiteLLM
- [x] Service `litellm` dans `docker-compose.yml` (container, healthcheck, port 4000)
- [x] `litellm_config.yaml` — 4 modèles via OpenRouter (Gemini Flash, Gemini Lite, Embeddings, Claude)
- [x] Env vars définies (`LITELLM_MASTER_KEY`, `OPENROUTER_API_KEY`, `LLM_BASE_URL`)
- [x] `kg-worker` dépend de `litellm` (condition: service_healthy)
- [x] Proxy testé en standalone — health check + appel LLM + embeddings (notebook 01)

### 1.2 Valider OrpheamRouter via le proxy
- [x] `OrpheamRouter` utilise `api_base` = gateway LiteLLM
- [x] `custom_llm_provider="openai"` + `api_key` ajoutés dans router.py
- [x] Appel via `OrpheamRouter` → proxy → OpenRouter validé (notebook 01)

### 1.3 Valider Graphiti via le proxy
- [x] `OrpheamGraphitiClient` refactoré — provider-agnostique via LiteLLM
- [x] `_ProxyCompatibleOpenAIClient` — contourne `/v1/responses` → `/v1/chat/completions`
- [x] `OrpheamSearchConfig` — config de recherche décomposée avec recettes pré-configurées
- [x] Notebook 03 validé (build indices + add_episode + search)

---

## Sprint 2 — Pipeline KG ✅

### 2.1 Rework parsers → orpheam-libs ✅
- [x] `BaseParser` — interface abstraite dans `orpheam/graph/parsers/base.py`
- [x] `EpisodeRecord` — modèle partagé dans `orpheam/graph/parsers/models.py`
- [x] `CsvParser` migré dans la lib (détection encoding, batches configurables)
- [x] `PdfParser` migré dans la lib (page par page, détection tables)
- [x] `ParserRegistry` — registre extensible (détection auto par extension)
- [x] Imports mis à jour dans ia/workers, anciens parsers supprimés
- [x] 20/20 tests unitaires passent

### 2.2 Pipeline orchestration → orpheam-libs ✅
- [x] Steps (parse → ingest → report) migrés dans `orpheam/graph/pipeline/`
- [x] `build_kg_pipeline()` — assemblage LangGraph dans la lib
- [x] `BatchOrchestrator` — traitement multi-fichiers dans la lib
- [x] Workers = re-exports + Celery tasks (glue uniquement)
- [x] Params morts supprimés (`use_v3`, `file_format`, `deduplicate`, `infer_cross_relations`)
- [x] 20/20 tests unitaires passent
- [x] Notebook 04 — test complet de la stack lib

### 2.3 Audit Graphiti vs étapes custom ✅
> Pipeline V3 actuel : Parsing → Chunking → Embedding → NER → Extraction → Transformation → Enrichment → Validation → Storage

- [x] Mapper ce que Graphiti gère nativement (extraction, dedup, entity resolution, temporalité, storage)
- [x] Identifier les étapes à wrapper vs remplacer vs compléter
- [x] Décider pour chaque feature : wrap Graphiti ou implémentation custom

**Résultat** : Graphiti gère ~85% du pipeline nativement (chunking, embedding, NER, extraction, dedup fuzzy, enrichissement communities/sagas, storage). Code custom nécessaire uniquement pour : parsing fichiers, orchestration LangGraph, entity linking externe (Wikidata/DBpedia). Architecture validée — pas de refonte nécessaire.

### 2.4 Améliorations pipeline (validé par audit) ✅
- [x] **Multi-Format Support** — TxtParser, JsonParser, XlsxParser, DocxParser ajoutés dans orpheam-libs
- [x] ~~Hierarchical Chunking~~ — géré nativement par Graphiti
- [x] ~~Multi-Pass Extraction~~ — géré nativement par Graphiti (entity resolution LLM)
- [x] ~~Graph-Aware Extraction~~ — géré nativement par Graphiti (dedup fuzzy + entity resolution)
- [x] 33 tests parsers passent (CSV, PDF, TXT, JSON, XLSX, DOCX + registry auto-detect)
- [x] Notebook 04 mis à jour avec tests des nouveaux parsers

### 2.5 Configurabilité & Évaluation
- [x] **KGPipelineConfig** — objet centralisé pour tous les paramètres tunables du pipeline
- [x] **Guide d'évaluation** — `docs/Evaluation Pipeline KG - Guide.md` (méthodes, métriques, benchmarks A/B)
- [ ] **Gold standard** — annoter 30-50 chunks (entités/relations attendues) pour baseline F1
- [ ] **Notebook benchmark** — comparer configs A/B (modèles, batch sizes, search modes)
- [ ] **Dataset retrieval** — 30-50 questions + réponses attendues pour évaluer la recherche

### 2.6 Optimisation pipeline ⬅️ PROCHAINE ÉTAPE
> Prérequis : avoir une baseline d'évaluation (2.5) avant d'optimiser
- [ ] **Cache LLM** — Redis cache dans LiteLLM (remplacer cache local)
- [ ] **Async ingestion** — paralléliser les appels `add_episode` (actuellement séquentiel)
- [ ] **Retry & circuit breaker** — gestion des erreurs LLM transitoires
- [ ] **Token budget** — tracking et limites de coût par pipeline run
- [ ] **Batch embedding** — grouper les appels d'embedding pour réduire la latence
- [ ] **Monitoring** — métriques de performance par étape (temps, tokens, coût)

---

## Sprint 3 — Graph Data Science (Analyse)

### 3.1 Intégration Neo4j GDS
- [ ] **Centralité** — identifier les entités les plus connectées/importantes
- [ ] **Communautés** — détecter les clusters d'entités liées
- [ ] **Similarité structurelle** — trouver des entités avec patterns de relations similaires
- [ ] **Chemins** — calculer les connexions entre deux entités

### 3.2 Combinaison KG + Embeddings + GDS
- [ ] Multi-Factor Ranking (centralité + similarité sémantique)
- [ ] Enrichir le contexte RAG avec les résultats GDS
- [ ] Notebook Jupyter de démonstration

---

## Sprint 4 — Refonte architecture (si nécessaire)
- [ ] Évaluer l'archi après sprints 1-3
- [ ] Identifier les bottlenecks ou couplages
- [ ] Refactorer si besoin

---

## Notes
- **Méthodo** : coder dans la lib → mettre à jour workers → `make test` → notebook → commit → docs
- **Règle** : modifier la lib (`orpheam-libs`) en priorité, le code prod (`ia/workers`) seulement si spécifique au domaine
- **Tests/Docs** : Jupyter notebooks dans le dossier test
- **LLM en dev** : switcher facilement de modèle via OpenRouter + proxy LiteLLM
- **Multi-tenant** : les clés API users viendront de Laravel (sprint futur)
