# TODO - Sprint IA

## Sprint 1 — Intégration LiteLLM

### 1.1 Configuration du proxy LiteLLM
- [x] Service `litellm` dans `docker-compose.yml` (container, healthcheck, port 4000)
- [x] `litellm_config.yaml` — 4 modèles via OpenRouter (Gemini Flash, Gemini Lite, Embeddings, Claude)
- [x] Env vars définies (`LITELLM_MASTER_KEY`, `OPENROUTER_API_KEY`, `LLM_BASE_URL`)
- [x] `kg-worker` dépend de `litellm` (condition: service_healthy)
- [x] Proxy testé en standalone — health check + appel LLM + embeddings (notebook 01)
- [ ] Ajouter des modèles supplémentaires au config si besoin (GPT-4o, etc.)

### 1.2 Valider OrpheamRouter via le proxy
- [x] `OrpheamRouter` utilise `api_base` = gateway LiteLLM
- [x] Fallback chain, token tracking, cost estimation, streaming implémentés
- [x] Appel via `OrpheamRouter` → proxy → OpenRouter validé (notebook 01)
- [x] `custom_llm_provider="openai"` + `api_key` ajoutés dans router.py
- [ ] Valider la fallback chain à travers le proxy
- [ ] Support clé API dynamique (futur multi-tenant — reporté)

### 1.3 Valider les workers via le proxy
- [x] `get_graphiti_client()` lit `LLM_BASE_URL` (= `http://litellm:4000/v1` via compose)
- [x] Env var fallback corrigé (plus de bypass OpenRouter direct)
- [ ] Tester KG pipeline end-to-end via le proxy
- [ ] Tester switch de modèle en dev (changer `GRAPHITI_LLM_MODEL` et relancer)

---

## Sprint 2 — Amélioration Pipeline KG

### 2.1 Intégration Graphiti via LiteLLM
- [x] `OrpheamGraphitiClient` refactoré — plus de dépendance Gemini directe
- [x] `_ProxyCompatibleOpenAIClient` — override `/v1/responses` → `/v1/chat/completions` + JSON mode
- [x] `neo4j_service.py` mis à jour (env vars `LITELLM_MASTER_KEY`, `LLM_BASE_URL`, modèles)
- [x] `OrpheamSearchConfig` — config de recherche décomposée et configurable
  - Recettes pré-configurées : `hybrid_rrf()`, `hybrid_cross_encoder()`, `hybrid_mmr()`, `edges_only()`, `nodes_only()`
  - `search()` unifié — accepte un `config` optionnel
- [x] `OpenAIRerankerClient` vérifié — utilise `/v1/chat/completions` (pas de problème proxy)
- [ ] **Tester notebook 03** — build indices + add_episode + search (en cours)
- [ ] Valider la recherche avec différentes configs (`OrpheamSearchConfig`)
- [ ] Notebook Jupyter de validation end-to-end

### 2.2 Audit Graphiti vs étapes custom
> Pipeline V3 actuel : Parsing → Chunking → Embedding → NER → Extraction → Transformation → Enrichment → Validation → Storage

- [ ] Mapper ce que Graphiti gère déjà nativement (extraction, dedup, entity resolution, temporal validity, storage Neo4j)
- [ ] Identifier les étapes à wrapper vs remplacer vs compléter
- [ ] Décider pour chaque feature : wrap Graphiti ou implémentation custom

### 2.3 Améliorations pipeline (à valider selon audit)
- [ ] **Multi-Pass Extraction** — raffiner entités/relations en plusieurs passes
- [ ] **Graph-Aware Extraction** — contexte du graphe existant pour éviter doublons
- [ ] **Hierarchical Chunking** — chunking conscient de la structure (sections, titres)
- [ ] **Incremental Entity Resolution** — fusion des mentions similaires en temps réel
- [ ] **Multi-Format Support** — étendre parsers (XLSX, XML si pas déjà fait)
- [ ] **Batch Processing** — déduplication cross-fichiers
- [ ] Notebooks Jupyter de test pour chaque amélioration

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
- **Règle** : modifier la lib (`orpheam-libs`) en priorité, le code prod (`ia/workers`) seulement si spécifique au domaine
- **Tests/Docs** : Jupyter notebooks dans le dossier test
- **LLM en dev** : switcher facilement de modèle via OpenRouter + proxy LiteLLM
- **Multi-tenant** : les clés API users viendront de Laravel (sprint futur)
