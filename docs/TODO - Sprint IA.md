# TODO - Sprint IA

> Organisation par **Phases**. Chaque phase correspond à un axe majeur du projet.

---

## Phase 1 — Promethee : Pipeline KG ✅ (en finalisation)

> Objectif : pipeline fiable et mesuré pour construire des Knowledge Graphs à partir de documents.

### 1.1 Intégration LiteLLM ✅
- [x] Service `litellm` dans `docker-compose.yml` (container, healthcheck, port 4000)
- [x] `litellm_config.yaml` — 4 modèles via OpenRouter (Gemini Flash, Gemini Lite, Embeddings, Claude)
- [x] `OrpheamRouter` utilise `api_base` = gateway LiteLLM
- [x] `OrpheamGraphitiClient` refactoré — provider-agnostique via LiteLLM
- [x] Proxy testé en standalone (notebook 01)

### 1.2 Pipeline KG ✅
- [x] Rework parsers → orpheam-libs (BaseParser, ParserRegistry, 6 formats)
- [x] Pipeline orchestration → orpheam-libs (LangGraph steps, BatchOrchestrator)
- [x] Audit Graphiti vs étapes custom — Graphiti gère ~85% nativement
- [x] Multi-format : CSV, PDF, TXT, JSON, XLSX, DOCX + registry auto-detect
- [x] 33 tests parsers passent
- [x] KGPipelineConfig — objet centralisé pour tous les paramètres tunables

### 1.3 Pipeline V3 Neuro-Symbolique ✅ (Laz)
- [x] Chemin classique (CSV/JSON) — 4 nœuds : Entity → Relation → Validation → Storage
- [x] Chemin neuro-symbolique (Markdown) — 7 nœuds : Markdown → SCT → Entity → Relation → ASP → Validation → Storage
- [x] SCT (GNN + Adaptive Fusion), ASP (clingo + réparation auto)
- [x] Forced JSON mode → 100% de parsing réussi
- [ ] Tests d'intégration pipeline V3 dans la nouvelle archi
- [ ] Notebook dédié pipeline V3

### 1.4 Parser Worker (file2md) ✅
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

### 1.5 Benchmark & Évaluation
- [x] Guide d'évaluation — `docs/Evaluation Pipeline KG - Guide.md`
- [ ] Gold standard — 10-15 documents annotés (entités + relations attendues)
- [ ] Métriques extraction : Precision, Recall, F1 (exact + fuzzy + sémantique)
- [ ] Métriques retrieval : Hit Rate, MRR, Precision@K, RAGAS
- [ ] Benchmark A/B — comparer modèles LLM, modes de recherche, embeddings
- [ ] Notebook benchmark avec tableau comparatif + export CSV

### 1.6 Optimisation Pipeline
- [ ] Cache LLM Redis via LiteLLM
- [ ] Async ingestion — paralléliser `add_episode()`
- [ ] Retry + circuit breaker sur erreurs LLM
- [ ] Token budget — tracking et limites de coût par run
- [ ] Batch embedding
- [ ] Monitoring par étape (Flower/Grafana)

### 1.7 Graph Data Science (Neo4j GDS)
- [ ] Centralité — entités les plus connectées/importantes
- [ ] Communautés — clusters d'entités (Louvain, Leiden)
- [ ] Similarité structurelle — patterns de relations similaires
- [ ] Chemins — connexions entre deux entités
- [ ] Multi-Factor Ranking (centralité + similarité sémantique)
- [ ] Notebook de démonstration GDS

---

## Phase 2 — Apollon, Muses & Pneuma : Orchestrateur Multi-Agents

> Objectif : orchestrateur intelligent (Apollon) qui route vers des sub-agents configurables (Muses),
> équipés de skills MCP (Pneuma) et connectés à des modules de graphs (Promethee).
>
> **Architecture cible** :
> ```
> User → Apollon (orchestrateur)
>            ├── Muse A (LLM X + skills Pneuma [web, calc] + Graph module juridique)
>            ├── Muse B (LLM Y + skills Pneuma [code, search] + Graph module technique)
>            └── Muse C (LLM Z + skills Pneuma [vision] + Graph module médical)
> ```
>
> Une **Muse** = choix LLM + ensemble de skills MCP (Pneuma) + modules de graphs (petits ou gros, par catégorie).
> La gestion des catégories de Muses se fait côté web (Laravel).

### 2.1 Refonte Apollon — Orchestrateur Central
- [ ] Remplacer Apollon legacy (LangGraph monolithique) par `OrpheamSupervisor`
- [ ] Routage dynamique : analyse de la requête → sélection de la Muse appropriée
- [ ] Gestion de contexte : mémoire partagée entre Apollon et les Muses (Redis checkpointer)
- [ ] Multi-turn : conversation continue, Apollon maintient l'état global
- [ ] Fallback : si une Muse échoue, Apollon reroute vers une autre

### 2.2 Muses — Sub-Agents Configurables
- [ ] Interface `Muse` — agent configurable : LLM + skills + graphs
- [ ] Configuration dynamique : une Muse est définie par un JSON/config (pas de code)
- [ ] Registre de Muses — discovery et instanciation à la demande
- [ ] Chaque Muse connectée à un ou plusieurs modules de graphs Promethee
- [ ] A2A (Agent-to-Agent) — communication inter-Muses avec circuit breaker
- [ ] Lifecycle hooks : init, pre-execute, post-execute, cleanup

### 2.3 Pneuma — Skills MCP
- [ ] Registre MCP centralisé — skills disponibles pour toutes les Muses
- [ ] Skills de base : KG search, file parsing, web search, calculator
- [ ] Intercepteurs MCP : logging, rate limiting, auth, cache
- [ ] Skills composables : un skill peut appeler d'autres skills
- [ ] Hot-reload : ajouter/retirer des skills sans redémarrer
- [ ] SDK skill : template pour créer un nouveau skill MCP rapidement

### 2.4 Computer Vision
- [ ] Intégration vision pour documents complexes (layouts, diagrammes, schémas)
- [ ] OCR avancé : tableaux imbriqués, formules mathématiques, handwriting
- [ ] Extraction structurée depuis images → entités/relations KG
- [ ] Pipeline multimodal : texte + images + tableaux dans un document
- [ ] Benchmark vision models via OpenRouter (GPT-4V, Claude Vision, Gemini Pro Vision)

### 2.5 Optimisation LLM
- [ ] Routing intelligent : modèle léger (tâches simples) vs lourd (extraction complexe)
- [ ] Chaîne de fallback adaptive (coût vs qualité vs latence)
- [ ] Few-shot / prompts spécialisés par domaine
- [ ] Distillation : modèle local entraîné sur outputs des gros modèles

### 2.6 Intégration Laravel
- [ ] API REST/WebSocket pour créer/configurer des Muses depuis le web
- [ ] Dashboard : état des Muses, graphs connectés, skills actifs
- [ ] Multi-tenant : isolation par user/organisation
- [ ] Clés API users gérées côté Laravel

### 2.7 Mise en Prod & Kubernetes
- [ ] Docker images multi-stage optimisés (CPU vs GPU)
- [ ] Healthchecks — endpoints `/health` pour chaque worker
- [ ] Logs structurés JSON (Loki, ELK)
- [ ] CI/CD — GitHub Actions : lint + tests sur PR, build Docker sur merge
- [ ] Kubernetes — déploiement K8s (voir 2.8)

### 2.8 Infrastructure Kubernetes
> K8s ouvre des possibilités impossibles en Docker Compose simple.

#### Neo4j Causal Clustering — Graphes distribués haute dispo
- [ ] Cluster Neo4j (3+ instances) : 1 Leader (écriture) + N Followers (lecture)
  - Read replicas pour scaling horizontal des requêtes GDS et search
  - Failover automatique : si le Leader tombe, un Follower prend le relais
  - Réplication synchrone : cohérence forte des données
- [ ] Multi-DB distribué — une base par domaine/client, chaque DB sur le cluster
- [ ] Neo4j Fabric — requêtes fédérées cross-DB (ex: "cherche dans tous les graphs juridiques")
- [ ] Helm chart Neo4j pour déploiement K8s standardisé

#### Serving de graphes distribués
- [ ] **Graph API Gateway** — point d'entrée unique pour tous les graphs
  - Routing par `graph_id` vers l'instance Neo4j appropriée
  - Load balancing lecture/écriture (Leader vs Followers)
  - Cache distribué (Redis Cluster) devant les requêtes fréquentes
- [ ] **Config Registry** — mapping dynamique `graph_id` → connexion Neo4j
  - Stocké dans ConfigMap K8s ou etcd
  - Hot-reload : ajouter un graph sans redémarrer les workers
- [ ] **Multi-tenant isolation** — namespace K8s par organisation
  - Network policies : isolation réseau entre tenants
  - Resource quotas : limites CPU/RAM par tenant

#### Inference GNN on-demand
- [ ] **Serving GNN** — modèles PyG/ONNX derrière une API (Triton, TorchServe, ou FastAPI)
  - GPU pool partagé K8s (NVIDIA GPU Operator)
  - Auto-scaling : scale-to-zero quand pas de requêtes, scale-up à la demande
  - Batch inference : grouper les requêtes pour maximiser le throughput GPU
- [ ] **GNN as a Service** — les Muses appellent l'inference GNN via Pneuma (skill MCP)
  - Node classification on-demand (ex: "de quel type est cette entité ?")
  - Link prediction on-demand (ex: "quelles relations manquent ?")
  - Graph embedding on-demand (ex: "représente ce sous-graphe en vecteur")
- [ ] **Pipeline GNN** — entraînement périodique sur les KG mis à jour
  - CronJob K8s : ré-entraîner le GNN quand le graph évolue significativement
  - Model registry (MLflow ou simple MinIO) : versioning des modèles
  - A/B serving : comparer deux versions de modèle en production

#### Visualisation interactive de graphes
- [ ] **Frontend graph viewer** — visualisation interactive des KG
  - Technologies : Neo4j Bloom (natif), ou custom avec D3.js / Cytoscape.js / Sigma.js
  - Intégration Laravel : iframe ou composant Vue/React
  - Filtres dynamiques : par type d'entité, période, communauté, centralité
- [ ] **Exploration on-demand** — expansion de nœuds en temps réel
  - Click sur un nœud → charge ses voisins (lazy loading)
  - Highlight communautés (Louvain) en couleurs
  - Taille des nœuds proportionnelle à la centralité (PageRank)
  - Timeline slider : voir l'évolution temporelle du graph (Graphiti)
- [ ] **Graph Studio** — outil d'analyse pour les data scientists
  - Exécuter des algos GDS depuis l'UI (centralité, communautés, chemins)
  - Annoter manuellement des entités/relations (feedback loop → améliorer le pipeline)
  - Export sous-graphes (GraphML, JSON, CSV) pour analyse externe

#### Graph on-demand — Création dynamique de graphes
- [ ] **Ephemeral graphs** — graphs temporaires créés à la volée pour une requête
  - User upload un document → Promethee crée un graph temporaire → Muse l'interroge → graph détruit
  - Namespace K8s éphémère ou Neo4j in-memory (APOC Virtual Graphs)
  - Cas d'usage : analyse ponctuelle d'un contrat, audit d'un rapport
- [ ] **Graph composition** — fusionner/découper des graphs à la demande
  - Merge : combiner graph juridique + graph technique pour une analyse croisée
  - Split : extraire un sous-graphe thématique depuis un graph généraliste
  - Snapshot : figer l'état d'un graph à un instant T (branching)
- [ ] **Self-service graph** — les users créent leurs propres graphs depuis le web
  - Upload documents → choix du parser → choix du LLM → graph personnel
  - Partage : rendre un graph accessible à d'autres users/organisations
  - Quotas : limites par user (nombre de graphs, taille, requêtes/jour)

---

## Phase 3 — Graph Data Science : Structures, Algorithmes & Quantum

> Objectif : exploiter la science des graphes pour maximiser la valeur des KG construits par Promethee.
> Trois axes complémentaires : structures avancées, algorithmes classiques, et calcul quantique.
>
> Détails techniques dans `docs/Plan.md`.

### 3.1 Neo4j GDS — Algorithmes classiques
- [ ] **Centralité** — PageRank, Betweenness, Degree, Eigenvector sur les KG Orpheam
- [ ] **Communautés** — Louvain, Leiden, Label Propagation
- [ ] **Similarité** — Node Similarity, Jaccard, Cosine structurelle
- [ ] **Chemins** — Shortest Path, All Shortest Paths, Dijkstra pondéré
- [ ] **Link Prediction classique** — Common Neighbors, Adamic-Adar, Preferential Attachment
- [ ] Multi-Factor Ranking (centralité + similarité sémantique + temporalité)
- [ ] Notebook de démonstration GDS
- [ ] Intégration dans les Muses comme source de contexte enrichi

### 3.2 Multi-Layer Graphs — Structures avancées
> Un graphe multi-couches sépare les relations par type/domaine sur des layers distincts,
> permettant des analyses croisées impossibles sur un graphe plat.

- [ ] **Architecture multi-layer** — design du schéma Neo4j (labels = layers, projections GDS par layer)
- [ ] **Kubernetes orchestration** — un namespace/deployment par layer de graph
  - Config registry : mapping `graph_id` → connexion Neo4j (host, port, credentials)
  - Routing Celery : queue par layer, workers dédiés par graph
  - Auto-scaling : HPA basé sur la taille du graph et la charge
- [ ] **Types de layers** à tester :
  - Layer sémantique (entités + relations extraites par LLM)
  - Layer structurel (hiérarchie documents, sections, paragraphes)
  - Layer temporel (évolution des entités dans le temps, Graphiti)
  - Layer externe (Wikidata, DBpedia, liens enrichis)
- [ ] **Cross-layer queries** — requêtes traversant plusieurs layers
  - Projections multi-graphes GDS
  - Fusion de résultats inter-layers (union, intersection, pondération)
- [ ] **Multi-DB Neo4j** — une base par domaine/client (isolation multi-tenant)
  - Neo4j Fabric pour requêtes fédérées cross-DB
  - Ou multiple instances Neo4j derrière un routeur
- [ ] Notebook multi-layer : créer 2-3 layers, exécuter des queries croisées
- [ ] Benchmark : graph plat vs multi-layer (qualité retrieval, latence)

### 3.3 LightRAG — RAG léger complémentaire
> Approche RAG légère et performante, complémentaire au pipeline KG.

- [ ] Benchmark LightRAG vs pipeline actuel (qualité retrieval, latence, coût)
- [ ] POC notebook avec dataset existant
- [ ] Décision : complémentaire (en parallèle de Graphiti) ou remplacement
- [ ] Intégration comme source de contexte alternative dans les Muses
- [ ] Évaluation RAGAS comparée au KG Graphiti

### 3.4 GNN classiques — Graph Neural Networks
> Baseline classique avant le quantum. PyTorch Geometric (PyG) pour du message passing sur GPU.

- [ ] POC GNN sur sous-graphe Neo4j (node classification, link prediction)
- [ ] Comparer GCN, GAT, GraphSAGE sur les KG Orpheam
- [ ] Benchmark vs algorithmes GDS natifs (qualité, vitesse)
- [ ] Évaluer si le SCT de Laz (LightweightGNN) suffit ou si PyG apporte un gain

### 3.5 QNN — Quantum Graph Neural Networks
> Point de convergence KG + Quantum Computing.
> Les graphes s'encodent naturellement sur circuits quantiques (nœuds → qubits, arêtes → intrication).

#### Recherche
- [ ] Revue de littérature : QGNN, Quantum Walk, VQE, QAOA pour graphes
- [ ] Fiches Obsidian (`recherche/papers/`) : papiers clés
- [ ] Évaluer frameworks : PennyLane, Qiskit, Cirq, TensorFlow Quantum

#### Accès au calcul quantique
| Fournisseur | Accès | Qubits | Coût estimé |
|-------------|-------|--------|-------------|
| **IBM Quantum** | Qiskit Runtime, plan gratuit (simulateurs) + pay-as-you-go (QPU) | 127-1121 (Eagle, Heron) | ~1.60$/sec QPU |
| **Amazon Braket** | AWS, on-demand | IonQ (36), Rigetti (84), IQM | ~0.30$/task + shot pricing |
| **Google Quantum AI** | Cirq, accès recherche | Sycamore (72) | Programme recherche |
| **Azure Quantum** | Credits gratuits ($500), IonQ/Quantinuum | Variable | Pay-per-job |
| **IonQ** | Direct API, Qiskit/Cirq/Braket | Forte (36 qubits) | $0.01/gate shot |
| **Simulateurs locaux** | PennyLane `default.qubit`, Qiskit Aer | ~25-30 qubits (RAM) | Gratuit |

> **Stratégie** : prototyper sur simulateur local (PennyLane), valider sur IBM Quantum (gratuit),
> scaler sur Amazon Braket ou Azure Quantum pour les vrais benchmarks.

#### Prototypage
- [ ] Simulateur local PennyLane (`default.qubit`, ~25 qubits)
- [ ] Sous-graphe test extrait de Neo4j (100-500 nœuds)
- [ ] POC détection de communautés — QAOA vs Louvain classique
- [ ] POC link prediction — QGNN variationnel vs GNN classique (PyG)
- [ ] Notebook `quantum_graph_poc.ipynb`

#### Cas d'usage
- [ ] Détection de communautés massives (QAOA / MaxCut, superposition 2^n partitions)
- [ ] Similarité structurelle (Quantum Walk, analogies inter-domaines)
- [ ] Requêtes multi-hop (Grover O(√N) vs O(N) classique)
- [ ] Quantum PageRank + VQE multi-objectifs
- [ ] Prédiction de liens (espace de Hilbert exponentiel)

#### Intégration Orpheam
- [ ] Module `orpheam/quantum/` dans orpheam-libs
- [ ] Interface unifiée : même API que GDS classique, backend quantum transparent
- [ ] Hybrid pipeline : pré-traitement classique → calcul quantique → post-traitement
- [ ] Routage auto : classique (petits graphes) vs quantum (graphes massifs)
- [ ] Accès cloud QPU (IBM Quantum Runtime, Amazon Braket)

---

## Prochaines étapes immédiates (Phase 1 — finalisation)

- [ ] **Build + test parse-worker Docker** — `make build-worker && make up-worker`
- [ ] **Notebook 07 E2E** — test complet parse-worker Docker → MinIO → KG
- [ ] **Mettre à jour notebook 06** — sorties benchmark-ready
- [ ] **Zotero + Obsidian** — configurer plugin, fiches de lecture GDS
- [ ] **Gold standard** — commencer annotation des documents de test
- [ ] **Laravel ↔ MinIO ↔ RabbitMQ** — côté Laravel pour mise en prod

---

## Notes

- **Nommage** : Promethee (graphs), Apollon (orchestrateur), Muses (sub-agents), Pneuma (skills MCP)
- **Méthodo** : coder dans la lib → mettre à jour workers → `make test` → notebook → commit → docs
- **Règle** : modifier la lib (`orpheam-libs`) en priorité, le code prod (`ia/workers`) seulement si spécifique au domaine
- **Tests/Docs** : Jupyter notebooks dans le dossier test
- **LLM en dev** : switcher facilement de modèle via OpenRouter + proxy LiteLLM
- **Pipeline V3** : développé par Laz, intégré dans la nouvelle archi — doc dans `docs/promethee/pipeline-v3/`
