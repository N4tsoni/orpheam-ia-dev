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

### 2.6 Parser Worker dédié — file2md
> Le parsing est le point d'entrée du pipeline. Il doit être isolé, performant et scalable indépendamment.
> Code source : `file2md/` — logique de parsing hybride Marker + MarkItDown.
> Décision archi : MinIO (S3) + Option A (Job Laravel + RabbitMQ). Détails dans `docs/promethee/Parser Worker - Options Integration.md`.

#### Logique de parsing (existant, à réutiliser)
- [x] **Parsing hybride PDF** — analyse de complexité par page → trivial (MarkItDown, <100ms/page) vs complexe (Marker, vision+OCR batch)
- [x] **MarkItDown** — parser léger pdfminer, stateless, parallélisable (pages triviales)
- [x] **Marker** — parser PyTorch avec vision + OCR, pool de modèles partagé (pages complexes : scans, tableaux, layouts)
- [x] **Multi-format** — PDF, DOCX, EPUB, Markdown, TXT
- [x] **Post-traitement** — sanitize (PUA, control chars), normalisation typographique, reconstruction tables

#### Étape 1 — Infra MinIO ✅
- [x] **Ajouter MinIO** dans `docker-compose.yml` (prod) et `docker-compose.test.yml` (dev)
- [x] **Config MinIO** — env vars dans `.env`, `.env.example`, `docker-compose.yml`
- [x] **Client S3 Python** — `infra/s3_client.py` (download, upload, ensure_bucket)
- [x] **Config Settings** — MinIO dans `core/config.py` (endpoint, credentials, bucket, ssl)
- [ ] **Côté Laravel** — configurer `Storage::disk('s3')` avec endpoint MinIO

#### Étape 2 — Celery parse-worker ✅
- [x] **Module `orpheam_workers/parsing/`** — `converter.py`, `complexity.py`, `sanitize.py`, `normalize.py`, `tables.py`, `pipeline.py`
- [x] **Task `process_parsing`** — download MinIO → parse → upload Markdown → résultat Laravel
- [x] **Celery worker** — `parse-worker` dans docker-compose, queue `parsing`
- [x] **Queue + routing** — queue `parsing` dans `celery_app.py`, routing dans `dispatch.py`
- [ ] **Dépendances** — `marker-pdf`, `markitdown`, `minio` dans `pyproject.toml` (group `parsing`)

#### Étape 3 — Branchement Laravel → parse-worker ✅
- [x] **JobType.PARSING** + `ParsingPayload` dans les DTOs
- [x] **Consumer dispatch** — `PARSING → process_parsing` dans `dispatch.py`
- [x] **Résultat** — upload Markdown sur MinIO bucket `parsed/` + résultat vers Laravel
- [ ] **Job Laravel** — upload fichier sur MinIO (`putObject`) puis publie job RabbitMQ sur queue `parsing`

#### Étape 4 — Chaînage parse-worker → Pipeline V3 ✅
- [x] **Chaînage Celery** — flag `chain_to_kg` déclenche automatiquement `kg-extraction` via `send_task`
- [x] **S3FileResolver** — `file_source.py` gère le préfixe `s3://` pour télécharger depuis MinIO
- [ ] **Pipeline V3 Markdown path** — valider que la sortie alimente correctement `MarkdownParsingNode` → SCT
- [ ] **Adapter orpheam-libs** — la lib reçoit du Markdown prêt → simplifier `ParserRegistry`
- [ ] **Métadonnées** — propager complexité page, source format, confidence dans les `EpisodeRecord`

#### Étape 5 — Validation
- [ ] **Dépendances** — `marker-pdf`, `markitdown`, `minio`, `PyMuPDF` dans `pyproject.toml`
- [ ] **Tests unitaires** — parsing Marker, MarkItDown, sanitize, normalize
- [ ] **Tests E2E** — PDF → MinIO → parse-worker → Markdown → Pipeline V3 → Neo4j
- [ ] **Notebook** — démo du flux complet
- [ ] **Monitoring** — métriques parse-worker dans Flower

### 2.7 Optimisation pipeline
> Prérequis : avoir une baseline d'évaluation (2.5) avant d'optimiser
- [ ] **Cache LLM** — Redis cache dans LiteLLM (remplacer cache local)
- [ ] **Async ingestion** — paralléliser les appels `add_episode` (actuellement séquentiel)
- [ ] **Retry & circuit breaker** — gestion des erreurs LLM transitoires
- [ ] **Token budget** — tracking et limites de coût par pipeline run
- [ ] **Batch embedding** — grouper les appels d'embedding pour réduire la latence
- [ ] **Monitoring** — métriques de performance par étape (temps, tokens, coût)

---

## Sprint 2.8 — Pipeline V3 Neuro-Symbolique ✅ (Laz)

> Développé par **Lazare** sur `feat/pipeline-v3-improvements`, intégré via cherry-pick le 9 avril 2026.
> Code déplacé de `src/domain/kg_builder/pipeline_v3/` → `orpheam_workers/promethee/pipeline_v3/`

### Pipeline V3.1 — Deux chemins d'extraction
- [x] **Chemin classique** (CSV/JSON) — 4 noeuds : Entity → Relation → Validation → Storage
- [x] **Chemin neuro-symbolique** (Markdown) — 7 noeuds : Markdown → SCT → Entity → Relation → ASP → Validation → Storage
- [x] Sélection automatique du chemin selon le type d'entrée
- [x] Forced JSON mode → 100% de parsing réussi (vs ~60% avant)
- [x] Fallback séquentiel si LangGraph absent

### Extraction LLM améliorée
- [x] `EntityExtractionNode` — extraction avec JSON mode forcé + injection SCT
- [x] `RelationExtractionNode` — deux passes (explicite + implicite) + validation entity names
- [x] `ValidationNode` — dédup + cohérence (logique pure, pas de LLM)
- [x] `StorageNode` — batch storage Neo4j

### SCT — Semantic Condition Transformer (Laz)
- [x] `MarkdownParsingNode` — parsing structuré (headings, tables, code, listes, images)
- [x] `SCTNode` — GNN 1-couche + Adaptive Fusion → conditions sémantiques injectées dans les prompts
- [x] `NeighborhoodExtractor` — voisinage Neo4j batch (Cypher, profondeur 2)
- [x] `LightweightGNN` — SentenceTransformer (all-MiniLM-L6-v2) + mean aggregation
- [x] `AdaptiveFusion` — score cohérence + formatage textuel pour injection prompt
- [x] Graceful degradation (Neo4j absent → conditions vides, SentenceTransformer absent → hash embeddings)

### ASP — Answer Set Programming (Laz)
- [x] `ASPVerificationNode` — vérification logique post-extraction
- [x] `ASPTranslator` — entités/relations → faits ASP (normalisation, reverse mapping)
- [x] `ASPSolver` — wrapper clingo avec timeout 30s
- [x] `ASPRepairEngine` — réparation auto (reflexive, dangling, type conflict, schema mismatch)
- [x] 3 fichiers de règles : `base_integrity.lp`, `schema_enforcement.lp`, `domain_rules.lp`
- [x] Graceful degradation (clingo absent → pass-through)

### Parsers & Utils (Laz)
- [x] `MarkdownParser` — chunks structurés avec hiérarchie headings
- [x] `JSONResponseParser` — parsing robuste des réponses LLM
- [x] `EntityParser` / `RelationParser` — validation Pydantic
- [x] `CacheManager` — cache sémantique (similarity 0.95, TTL 24h, LRU)
- [x] `LLMClient` — client OpenRouter avec forced JSON mode

### Dépendances ajoutées
- [x] `clingo ^5.7.1` — solveur ASP
- [x] `networkx ^3.2.0` — manipulation graphes (GNN)
- [x] `mistune ^3.0.2` — parsing Markdown

### Intégration nouvelle archi (9 avril 2026)
- [x] Cherry-pick des 2 commits depuis `feat/pipeline-v3-improvements`
- [x] Déplacement vers `orpheam_workers/promethee/pipeline_v3/`
- [x] Imports corrigés (`src.core.config` → `orpheam_workers.core.config`, storage_node → neo4j_service)
- [x] Suppression anciens modèles Entity/Relation → dicts
- [ ] **Tests à écrire** — valider l'intégration dans la nouvelle archi
- [ ] **Notebook** — démo du pipeline V3 neuro-symbolique

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
- [x] Évaluer l'archi après sprints 1-3
- [x] Identifier les bottlenecks ou couplages
- [x] Refactorer si besoin
- [x] Remplacement Apollon legacy → orchestrateur MCP multi-agent

---

## Sprint 5 — Intégration LightRAG

> Sujet à part entière. LightRAG est une approche de Retrieval-Augmented Generation légère et performante qui complète/remplace le pipeline RAG classique.

### 5.1 Étude & Prototypage
- [ ] **Benchmark LightRAG vs pipeline actuel** — comparer qualité retrieval, latence, coût
- [ ] **POC** — notebook de test avec le dataset existant
- [ ] **Décision** — complémentaire (en parallèle de Graphiti) ou remplacement

### 5.2 Intégration
- [ ] **Module LightRAG** dans orpheam-libs ou workers (selon décision archi)
- [ ] **Indexation** — pipeline d'ingestion LightRAG (documents → index)
- [ ] **Recherche** — intégration dans Apollon comme source de contexte alternative
- [ ] **Évaluation** — métriques retrieval (RAGAS) comparées au KG Graphiti

---

## Notes
- **Méthodo** : coder dans la lib → mettre à jour workers → `make test` → notebook → commit → docs
- **Règle** : modifier la lib (`orpheam-libs`) en priorité, le code prod (`ia/workers`) seulement si spécifique au domaine
- **Tests/Docs** : Jupyter notebooks dans le dossier test
- **LLM en dev** : switcher facilement de modèle via OpenRouter + proxy LiteLLM
- **Multi-tenant** : les clés API users viendront de Laravel (sprint futur)
- **Pipeline V3** : développé par Laz, intégré dans la nouvelle archi — doc dans `docs/promethee/pipeline-v3/`
- **file2md** : parser worker standalone (FastAPI + MinIO), code dans `file2md/` — à intégrer dans le pipeline via RabbitMQ
