# Recap Sprint — Du cherry-pick Pipeline V3 a la Session 5

> Période : 3 avril → 12 avril 2026
> 5 sessions de dev (Sessions 1-5)
> 2 devs : N4 + Laz (Pipeline V3)

---

## TL;DR pour la Daily

On a une **pipeline KG fonctionnelle de bout en bout** : un fichier (PDF, DOCX, TXT...) entre via RabbitMQ, est parsé en Markdown (Marker/MarkItDown), ingéré dans Neo4j via Graphiti, et le résultat revient à Laravel. La Phase 1 (Promethee) est quasi finalisée. Il reste le build Docker du parse-worker et les tests E2E en container. La doc est restructurée en 6 phases pour la suite (Apollon, K8s, GDS, GNN, Quantum).

---

## Ce qui a été livré

### 1. Stack LLM — Proxy LiteLLM + Graphiti (Sessions 1-2)

- **LiteLLM Proxy** centralisé : tous les appels LLM/embeddings/reranking passent par `http://litellm:4000` → OpenRouter → providers (Gemini, Claude, GPT)
- **OrpheamRouter** refactoré : plus de fallback côté client (géré par le proxy)
- **OrpheamGraphitiClient** : client Neo4j/Graphiti provider-agnostique via LiteLLM
- **ProxyCompatibleOpenAIClient** : contournement `/v1/responses` → `/v1/chat/completions` + JSON mode (puis supprimé en Session 5, graphiti-core 0.3.21 le gère nativement)
- **OrpheamSearchConfig** : recettes de recherche pré-configurées (hybrid_rrf, hybrid_cross_encoder, hybrid_mmr)

### 2. Pipeline KG — Parsers + Orchestration (Sessions 2-3)

- **Parsers migrés dans orpheam-libs** : BaseParser, ParserRegistry, 6 formats (CSV, PDF, TXT, JSON, XLSX, DOCX), 33 tests
- **Pipeline simplifiée** : suppression des splitters (Graphiti gère le chunking via `add_episode()`), pipeline réduite à `ingest → report`
- **KGPipelineConfig** : objet centralisé pour tous les paramètres tunables
- **BatchOrchestrator** : traitement multi-fichiers

### 3. Pipeline V3 Neuro-Symbolique (Laz)

- Cherry-pick intégré dans la nouvelle archi
- Chemin classique (CSV/JSON) : 4 nœuds (Entity → Relation → Validation → Storage)
- Chemin neuro-symbolique (Markdown) : 7 nœuds (Markdown → SCT → Entity → Relation → ASP → Validation → Storage)
- SCT (GNN + Adaptive Fusion), ASP (clingo + réparation auto)
- Forced JSON mode → 100% de parsing réussi

### 4. Parse-Worker — file2md (Sessions 3-4)

- **Parsing hybride PDF** par page : MarkItDown (pages triviales, pdfminer) + Marker (pages complexes, OCR/vision PyTorch)
- **4 modes parser** : `auto` (hybride), `marker`, `markitdown`, `sanitize`
- **Infra MinIO** : S3 client, upload/download, buckets documents/parsed
- **Celery task** `process_parsing` : queue `parsing`, retry, chaînage vers kg-pipeline (`chain_to_kg`)
- **DTOs** : `JobPayload`, `ParsingPayload`, `JobResult` — contrat Laravel ↔ Workers
- **Docker** : `Dockerfile.parsing`, services dans `docker-compose.test.yml` (RabbitMQ + parse-worker standalone)
- **Makefile** : `up`, `up-worker`, `up-infra`, `build-worker`, `logs-worker`, `status`, `notebook`, `check`, `clean`

### 5. Documentation & Organisation (Sessions 4-5)

- **Restructuration Obsidian** : sous-dossiers thématiques (promethee/graphe, parsing, evaluation ; voice/stt, tts ; recherche → R&D)
- **TODOs éclatés en 6 phases** : Phase 1 Promethee, Phase 2 Apollon/Muses/Pneuma, Phase 3 K8s, Phase 4 GDS, Phase 5 GNN, Phase 6 Quantum
- **README enrichi** : flux, process de dev, commandes, architecture, roadmap
- **Architecture nommée** : Promethee (graphs), Apollon (orchestrateur), Muses (sub-agents), Pneuma (skills MCP)
- **Structure recherche** : templates papers/concepts/projets, notes Neo4j GDS, Graphiti, Centrality, Community Detection

### 6. Notebooks d'intégration

| Notebook | Contenu | Statut |
|----------|---------|--------|
| 01 — LiteLLM Proxy | Health, LLM, embeddings, OrpheamRouter | Done |
| 02 — KG Pipeline | Parse → ingest → report → Neo4j | Done |
| 03 — Graphiti LiteLLM | Indices, ingestion, search, SearchConfig | Done |
| 04 — Stack Lib | Parsers, Graphiti, search config | Done |
| 05 — FTP Ingestion | Parsers sur fichiers distants | Done |
| 06 — Parse Worker | MinIO + parser + "METTEZ ICI LE PATH" + Graphiti | Done |
| 07 — E2E Pipeline | Laravel → RabbitMQ → Celery → MinIO → KG | Done (hors Docker) |

---

## Etat actuel — Phase 1 Promethee

| Bloc | Statut |
|------|--------|
| LiteLLM Proxy | Done |
| Pipeline KG (parsers, Graphiti, orchestration) | Done |
| Pipeline V3 neuro-symbolique (Laz) | Intégré, tests à écrire |
| Parse-worker (code Python) | Done |
| Parse-worker (Docker build + E2E) | A faire |
| Chainage parse → KG | Done (code) |
| Côté Laravel (MinIO + RabbitMQ) | A faire |
| Benchmark & évaluation (gold standard) | A faire |

---

## Ce qui bloque / reste à faire

### Court terme (Sprint courant)
- [ ] `make build-worker` — premier build Docker du parse-worker
- [ ] Notebook 07 E2E complet — test en containers Docker
- [ ] Côté Laravel — `Storage::disk('s3')` + job RabbitMQ queue `parsing`

### Moyen terme
- [ ] Gold standard — 10-15 documents annotés pour benchmark
- [ ] Notebook benchmark — métriques Precision/Recall/F1 + comparaison modèles
- [ ] Tests Pipeline V3 dans la nouvelle archi
- [ ] Cache LLM Redis via LiteLLM

### Phase 2 — Apollon / Muses / Pneuma
- Architecture définie (docs), code pas encore commencé
- Apollon = orchestrateur, Muses = sub-agents (LLM + skills + graph), Pneuma = registre MCP

---

## Commits (depuis le cherry-pick)

### orpheam-libs (13 commits)
```
0e49261 fix: suppression ProxyCompatibleOpenAIClient + small_model
c647e48 LIBS UPDATE -> Parser passé en Worker
9291216 Lib updated
2944f87 .md compatibility For parser
26ba6f8 refactor(parsers+pipeline): stream support, top-level imports
6f5c779 Update: Parser and BaseParser object for Parsing Stage
e9045fe refactor: migration des parsers et du pipeline KG dans orpheam-libs
590d814 Parser stage
01822c3 Refacto: New class Search Config for Graphiti
26879f3 Graphiti config pour qu'il tape sur LightLLM proxy
ebd11db Rework: Objet Orpeam Router pour api
2364885 Delete mock test
21c89a8 (fix) - README.md
```

### ia/workers (13 commits)
```
8ec95af feat: docker-compose env-file + consumer robustesse + notebook 07 enrichi
bdb23fe feat: choix du parser + notebook 07 E2E + RabbitMQ/parse-worker docker
02670b0 Update for clean dev and testing -> Docker Volume MinIO
99e2bbe Update -> Makefile notebook, install-dev, deps
1c06701 Pyproject.toml et docker update + Docker pour Parser Worker
87ab8cc S3 File resolver
5e09946 Chainage parse-worker → kg-pipeline via chain_to_kg
8693478 Branchement task parsing sur pipeline + DTOs
4e66a95 Ajout config MinIO + client S3 partagé
cbd3de6 add data folder to gitignore
e99da27 Ajout parse-worker + MinIO (S3) — squelette
5c1e765 Integration cherry pick dans la new archi
a58a828 (feat) - add ast & neuro-symbolic link
```

### parent repo (7 commits)
```
bc871b0 docs: restructuration Obsidian sous-dossiers + TODOs 6 phases
51ffe3f docs: roadmap 3 phases + architecture Apollon/Muses/Pneuma
40e39d1 Docs + Pointeurs
d4b716d Documentation Archi + Separation
13684cc Update SubModule
ea5eb6c Docs
b928d06 Graphiti + LightLLM + Notebooks + Obsidian Documentation
```
