# Plan — Roadmap Technique Orpheam IA

> Vision long terme : convergence Knowledge Graphs + IA multi-agents + Quantum Computing pour du raisonnement sur graphes massifs.

---

## Phase 1 — Promethee : Pipeline KG (en finalisation)

### 1.1 Tests d'intégration parse-worker
- [ ] Build + démarrage parse-worker Docker (`make build-worker && make up-worker`)
- [ ] Notebook 07 E2E complet : Laravel (sim) → RabbitMQ → parse-worker → MinIO → KG
- [ ] Tester les 4 modes parser : `auto`, `marker`, `markitdown`, `sanitize`
- [ ] Valider le chaînage `chain_to_kg` : parse-worker → kg-pipeline automatique
- [ ] Tester des documents variés : PDF scanné, PDF textuel, DOCX, EPUB, MD, TXT
- [ ] Mesurer temps de parsing par format et par parser

### 1.2 Pipeline V3 neuro-symbolique (Laz)
- [ ] Tests d'intégration du chemin Markdown → SCT → Extraction → ASP → Storage
- [ ] Valider que la sortie parse-worker (Markdown) alimente correctement `MarkdownParsingNode`
- [ ] Benchmark SCT activé vs désactivé (impact sur qualité extraction)
- [ ] Benchmark ASP activé vs désactivé (impact sur cohérence du graphe)
- [ ] Notebook dédié pipeline V3

### 1.3 Qualité d'extraction
- [ ] Gold standard : 10-15 documents annotés manuellement (entités + relations attendues)
- [ ] Métriques automatisées : Precision, Recall, F1 (exact + fuzzy + sémantique)
- [ ] Benchmark A/B : comparer modèles LLM, modes de recherche, embeddings
- [ ] Tracking coût/token par run

### 1.4 Performance & scalabilité
- [ ] Cache LLM Redis via LiteLLM (remplacer cache local)
- [ ] Async ingestion — paralléliser `add_episode()` (actuellement séquentiel)
- [ ] Retry + circuit breaker sur erreurs LLM transitoires
- [ ] Token budget — limites de coût par pipeline run
- [ ] Batch embedding — grouper les appels pour réduire latence
- [ ] Monitoring par étape (temps, tokens, coût) dans Flower/Grafana

---

## Phase 2 — Apollon, Muses & Pneuma : Orchestrateur Multi-Agents

> **Apollon** = orchestrateur central, route les requêtes vers les sub-agents.
> **Muses** = sub-agents configurables. Chaque Muse = choix LLM + skills Pneuma + modules de graphs Promethee.
> **Pneuma** = registre MCP centralisé de skills disponibles pour les Muses.
>
> ```
> User → Apollon (orchestrateur)
>            ├── Muse A (LLM X + Pneuma [web, calc] + Graph juridique)
>            ├── Muse B (LLM Y + Pneuma [code, search] + Graph technique)
>            └── Muse C (LLM Z + Pneuma [vision] + Graph médical)
> ```

### 2.1 Refonte Apollon — Orchestrateur Central
- [ ] Remplacer Apollon legacy (LangGraph monolithique) par `OrpheamSupervisor`
- [ ] Routage dynamique : analyse requête → sélection Muse appropriée
- [ ] Gestion de contexte : mémoire partagée Apollon ↔ Muses (Redis checkpointer)
- [ ] Multi-turn : conversation continue, Apollon maintient l'état global
- [ ] Fallback : si une Muse échoue, Apollon reroute

### 2.2 Muses — Sub-Agents Configurables
- [ ] Interface `Muse` — agent configurable : LLM + skills + graphs
- [ ] Configuration dynamique : une Muse définie par JSON/config (pas de code)
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
- [ ] Intégration modèles vision pour parsing de documents complexes (layouts, diagrammes, schémas)
- [ ] OCR avancé : reconnaissance de tableaux imbriqués, formules mathématiques, handwriting
- [ ] Extraction structurée depuis images : organigrammes → entités/relations KG
- [ ] Pipeline multimodal : texte + images + tableaux dans un seul document
- [ ] Benchmark vision models : GPT-4V vs Claude Vision vs Gemini Pro Vision via OpenRouter

### 2.5 Optimisation LLM
- [ ] Routing intelligent : modèle léger pour tâches simples, modèle lourd pour extraction complexe
- [ ] Chaîne de fallback adaptive (coût vs qualité vs latence)
- [ ] Fine-tuning / few-shot : prompts spécialisés par domaine (juridique, technique, médical)
- [ ] Distillation : entraîner un modèle local sur les outputs des gros modèles pour réduire les coûts

### 2.6 Infrastructure Kubernetes

> K8s transforme Orpheam d'un ensemble de containers en une plateforme distribuée.
> Chaque service scale indépendamment, les graphs sont servis à la demande.

#### Neo4j Causal Clustering — Haute disponibilité

```
┌─ Neo4j Causal Cluster ────────────────────────┐
│                                                │
│   ┌──────────┐   sync    ┌──────────┐         │
│   │  Leader  │ ────────► │ Follower │         │
│   │ (write)  │           │  (read)  │         │
│   └──────────┘           └──────────┘         │
│       │ sync                                   │
│       ▼                                        │
│   ┌──────────┐           ┌──────────┐         │
│   │ Follower │           │ Read     │ ← HPA   │
│   │  (read)  │           │ Replica  │   scale  │
│   └──────────┘           └──────────┘         │
│                                                │
│   Failover auto : Leader tombe → Follower élu  │
└────────────────────────────────────────────────┘
```

- 1 Leader (écriture) + N Followers (lecture) + Read Replicas (scale horizontal)
- Réplication synchrone, failover automatique
- Helm chart Neo4j pour déploiement standardisé

#### Serving de graphes distribués

```
┌─ Graph API Gateway ───────────────────────────────────┐
│                                                        │
│  Request → Route by graph_id → Neo4j instance          │
│                                                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ graph-juri  │  │ graph-tech  │  │ graph-med   │   │
│  │ Neo4j DB 1  │  │ Neo4j DB 2  │  │ Neo4j DB 3  │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
│         │                │                │            │
│         └────── Neo4j Fabric ────────────┘            │
│                (cross-DB queries)                      │
└────────────────────────────────────────────────────────┘
```

- Config Registry dynamique : `graph_id` → connexion Neo4j (ConfigMap/etcd, hot-reload)
- Load balancing lecture/écriture (Leader vs Followers)
- Cache distribué (Redis Cluster) devant les requêtes fréquentes
- Neo4j Fabric pour requêtes fédérées cross-DB
- Multi-tenant : namespace K8s par organisation, network policies, resource quotas

#### Inference GNN on-demand

```
Muse → Pneuma (skill MCP "gnn_inference") → GNN Serving API → GPU Pool
                                                    │
                                              ┌─────┴──────┐
                                              │ TorchServe  │
                                              │ ou Triton   │
                                              │ (ONNX/PyG)  │
                                              └────────────┘
```

- Serving GNN : modèles PyG/ONNX derrière TorchServe ou Triton Inference Server
- GPU pool partagé K8s (NVIDIA GPU Operator), scale-to-zero
- GNN as a Service via skill Pneuma : node classification, link prediction, graph embedding on-demand
- Pipeline d'entraînement : CronJob K8s, model registry (MLflow/MinIO), A/B serving

#### Visualisation interactive de graphes

- Frontend : Neo4j Bloom (natif) ou custom D3.js / Cytoscape.js / Sigma.js
- Intégration Laravel : composant Vue/React embarqué
- Exploration on-demand : lazy loading des voisins, communautés en couleurs, taille = centralité
- Timeline slider : évolution temporelle du graph (Graphiti)
- Graph Studio : exécuter GDS depuis l'UI, annoter manuellement (feedback loop), export sous-graphes

#### Graph on-demand — Création dynamique

- **Ephemeral graphs** : upload document → graph temporaire → interrogation → destruction
  - Neo4j in-memory (APOC Virtual Graphs) ou namespace K8s éphémère
- **Graph composition** : merge, split, snapshot de graphs à la demande
- **Self-service** : users créent leurs graphs depuis le web (upload → parser → LLM → graph personnel)

---

## Phase 3 — Graph Data Science : Structures, Algorithmes & Quantum

> Exploiter la science des graphes à tous les niveaux — des algorithmes classiques Neo4j GDS
> jusqu'au calcul quantique — pour maximiser la valeur des KG construits par Promethee.

### 3.1 Neo4j GDS — Algorithmes classiques

Algorithmes disponibles nativement dans Neo4j Graph Data Science Library :

| Catégorie | Algorithmes | Application Orpheam |
|-----------|-------------|---------------------|
| **Centralité** | PageRank, Betweenness, Degree, Eigenvector, HITS | Entités les plus importantes du KG |
| **Communautés** | Louvain, Leiden, Label Propagation, WCC, SCC | Clusters thématiques dans le KG |
| **Similarité** | Node Similarity, Jaccard, Cosine, Overlap | Entités avec patterns de relations similaires |
| **Chemins** | Shortest Path, Dijkstra, A*, Yen's K | Connexions entre entités (raisonnement multi-hop) |
| **Link Prediction** | Common Neighbors, Adamic-Adar, Pref. Attachment | Relations manquantes dans le KG |
| **Embeddings** | FastRP, Node2Vec, GraphSAGE, HashGNN | Représentations vectorielles des nœuds |

- Multi-Factor Ranking : combiner centralité + similarité sémantique + temporalité
- Enrichir le contexte RAG des Muses avec les résultats GDS
- 3 modes d'exécution GDS : `stream` (lecture), `mutate` (en mémoire), `write` (persisté)

### 3.2 Multi-Layer Graphs — Structures avancées

> Un graphe multi-couches sépare les relations par type/domaine sur des layers distincts,
> permettant des analyses croisées impossibles sur un graphe plat.

#### Architecture

```
              ┌─────────────────────────────────────┐
              │         Multi-Layer Graph            │
              │                                     │
Layer 4       │  ┌─────────────────────────────┐    │
(Externe)     │  │ Wikidata · DBpedia · liens  │    │
              │  └──────────┬──────────────────┘    │
              │             │ cross-layer            │
Layer 3       │  ┌──────────┴──────────────────┐    │
(Temporel)    │  │ Évolution · Graphiti · sagas │    │
              │  └──────────┬──────────────────┘    │
              │             │                       │
Layer 2       │  ┌──────────┴──────────────────┐    │
(Structurel)  │  │ Documents · sections · méta  │    │
              │  └──────────┬──────────────────┘    │
              │             │                       │
Layer 1       │  ┌──────────┴──────────────────┐    │
(Sémantique)  │  │ Entités · relations · LLM   │    │
              │  └─────────────────────────────┘    │
              └─────────────────────────────────────┘
```

#### Kubernetes orchestration

```
┌─ K8s Cluster ──────────────────────────────────────┐
│                                                     │
│  ┌─ Namespace: graph-juridique ──┐                 │
│  │  Neo4j (5 replicas)           │                 │
│  │  kg-worker (HPA, GPU pool)    │                 │
│  │  parse-worker (CPU pool)      │                 │
│  └───────────────────────────────┘                 │
│                                                     │
│  ┌─ Namespace: graph-technique ──┐                 │
│  │  Neo4j (3 replicas)           │                 │
│  │  kg-worker (HPA)              │                 │
│  │  parse-worker (CPU pool)      │                 │
│  └───────────────────────────────┘                 │
│                                                     │
│  ┌─ Namespace: shared ───────────┐                 │
│  │  Config Registry (graph_id →  │                 │
│  │    Neo4j connection)          │                 │
│  │  RabbitMQ (routing per queue) │                 │
│  │  MinIO (shared storage)       │                 │
│  │  LiteLLM (LLM gateway)       │                 │
│  └───────────────────────────────┘                 │
└─────────────────────────────────────────────────────┘
```

- Config registry : mapping `graph_id` → connexion Neo4j (host, port, credentials)
- Routing Celery : queue par domaine, workers dédiés
- Auto-scaling HPA basé sur taille du graph et charge
- Neo4j Fabric pour requêtes fédérées cross-DB (multi-tenant)

#### Cross-layer queries

- Projections multi-graphes GDS (union de layers)
- Fusion de résultats inter-layers (pondération, intersection)
- Requêtes traversant layers : "Quelle entité (L1) a le plus évolué (L3) dans les documents récents (L2) ?"

### 3.3 LightRAG — RAG léger complémentaire

Approche RAG légère et performante, en parallèle ou remplacement du pipeline KG lourd :
- Benchmark LightRAG vs Graphiti (qualité retrieval, latence, coût)
- Si complémentaire : source de contexte alternative dans les Muses
- Si remplacement : migration progressive avec métriques RAGAS

### 3.4 GNN classiques — Graph Neural Networks

Baseline GPU classique avant d'explorer le quantum :

| Architecture | Force | Usage Orpheam |
|-------------|-------|---------------|
| **GCN** (Kipf 2017) | Simple, rapide | Baseline node classification |
| **GAT** (Veličković 2018) | Attention sur voisins | Poids adaptatifs par relation |
| **GraphSAGE** (Hamilton 2017) | Inductif, scalable | Nouveaux nœuds sans ré-entraîner |
| **GIN** (Xu 2019) | Expressivité maximale (WL-test) | Classification de sous-graphes |

- Comparer avec le SCT de Laz (LightweightGNN, all-MiniLM-L6-v2 + mean aggregation)
- PyTorch Geometric (PyG) pour l'implémentation
- Benchmark GNN vs GDS natif vs SCT (qualité, vitesse, RAM)

### 3.5 QNN — Quantum Graph Neural Networks

> **Point de convergence** entre deux révolutions technologiques :
> - **Knowledge Graphs** — représentation structurée, raisonnement symbolique
> - **Quantum Computing** — parallélisme quantique, superposition, intrication
>
> Les QGNN exploitent la structure naturelle des graphes sur circuits quantiques
> pour résoudre des problèmes intractables en classique sur des graphes massifs.

#### Architecture QGNN

```
Graphe classique                Circuit quantique
                                
  A ── B                        |0⟩ ─ RY(θ_A) ─ CNOT ─ RY(φ₁) ─ Mesure
  │    │          ──────►       |0⟩ ─ RY(θ_B) ─ ──●── ─ CNOT ─── Mesure
  C ── D                        |0⟩ ─ RY(θ_C) ────────── ──●── ─ Mesure
                                |0⟩ ─ RY(θ_D) ──────────────── ─ Mesure
                                
  Nœuds → qubits (RX/RY/RZ)
  Arêtes → intrication (CNOT/CZ)
  Attributs → paramètres variationnels (θ)
```

- Couches paramétrées (Ansatz) adaptées à la topologie du graphe
- Quantum message passing : information propagée via intrication
- Entraînement hybride : forward QPU + backward classique (parameter-shift rule)

#### Accès au calcul quantique

| Fournisseur | Accès | Qubits | Coût |
|-------------|-------|--------|------|
| **IBM Quantum** | Qiskit Runtime, gratuit (sim) + pay-as-you-go | 127-1121 (Eagle/Heron) | ~1.60$/sec QPU |
| **Amazon Braket** | AWS on-demand, IonQ/Rigetti/IQM | 36-84 | ~0.30$/task + shots |
| **Azure Quantum** | Credits gratuits ($500), IonQ/Quantinuum | Variable | Pay-per-job |
| **Google Quantum AI** | Cirq, programme recherche | 72 (Sycamore) | Recherche |
| **IonQ Direct** | API, compatible Qiskit/Cirq/Braket | 36 (Forte) | $0.01/gate shot |
| **Simulateurs** | PennyLane `default.qubit`, Qiskit Aer | ~25-30 (limité RAM) | Gratuit |

> **Stratégie** : prototyper local (PennyLane) → valider IBM Quantum (gratuit) → scaler Braket/Azure.

#### Cas d'usage Orpheam

| Problème | Méthode classique | Méthode quantique | Avantage |
|----------|-------------------|-------------------|----------|
| Communautés (millions de nœuds) | Louvain O(n·log²n) | QAOA MaxCut | 2^n partitions en superposition |
| Similarité structurelle | VF2, WL-test | Quantum Walk | Temps polynomial vs exponentiel |
| Requêtes multi-hop (3+ sauts) | BFS/DFS O(N) | Grover | O(√N) |
| Ranking multi-facteurs | Scoring séquentiel | Quantum PageRank + VQE | Optimisation multi-objectifs simultanée |
| Link prediction | GNN classique | QGNN variationnel | Espace de Hilbert exponentiel |

---

## Progression par phase

```
Phase 1 — Promethee (Pipeline KG)               ████████████████████░  ~90%
  └── Reste : build Docker, E2E, benchmark

Phase 2 — Apollon/Muses/Pneuma + K8s            ░░░░░░░░░░░░░░░░░░░░  0%
  ├── 2.1-2.5 Orchestrateur + agents + skills
  └── 2.6 K8s : Neo4j Cluster, GNN serving, visu, graph on-demand

Phase 3 — Graph Data Science                     ░░░░░░░░░░░░░░░░░░░░  0%
  ├── 3.1 GDS classique     — prêt dès Phase 1 finie
  ├── 3.2 Multi-Layer       — prérequis : K8s (Phase 2.6)
  ├── 3.3 LightRAG          — indépendant, peut démarrer tôt
  ├── 3.4 GNN classiques    — prérequis : dataset benchmark + GPU (K8s)
  └── 3.5 QNN Quantum       — recherche dès maintenant, proto après 3.4
```

> **Phase 1** = socle (pipeline fiable et mesuré).
> **Phase 2** = capacités (agents, vision, K8s) + infra distribuée.
> **Phase 3** = science des graphes, du classique au quantum.
> K8s (Phase 2.6) est un enabler pour Multi-Layer (3.2) et GNN serving (3.4).
> La recherche Phase 3 (lectures, Obsidian) peut démarrer en parallèle.

---

## Ressources clés

| Domaine | Ressource | Usage |
|---------|-----------|-------|
| GDS | Neo4j GDS Library | Algorithmes classiques (centralité, communautés, etc.) |
| Multi-Layer | Neo4j Fabric | Requêtes fédérées cross-DB |
| GNN | PyTorch Geometric (PyG) | GNN classiques (GCN, GAT, GraphSAGE) |
| LightRAG | LightRAG | RAG léger alternatif |
| QGNN | PennyLane | Framework quantum ML, simulateurs + QPU |
| QGNN | Qiskit | IBM Quantum, circuits, Runtime |
| QAOA | Farhi et al. 2014 | Base théorique QAOA pour MaxCut |
| Quantum Walk | Childs et al. 2003 | Speedup exponentiel par quantum walk |
| Quantum PageRank | Paparo & Martin-Delgado 2012 | PageRank quantique |
| K8s | Kubernetes HPA + namespaces | Multi-graph orchestration |
