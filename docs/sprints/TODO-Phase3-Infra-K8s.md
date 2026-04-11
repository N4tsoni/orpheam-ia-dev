# TODO Phase 3 — Infrastructure Kubernetes

> K8s ouvre des possibilités impossibles en Docker Compose simple.

---

## Neo4j Causal Clustering — Graphes distribués haute dispo
- [ ] Cluster Neo4j (3+ instances) : 1 Leader (écriture) + N Followers (lecture)
  - Read replicas pour scaling horizontal des requêtes GDS et search
  - Failover automatique : si le Leader tombe, un Follower prend le relais
  - Réplication synchrone : cohérence forte des données
- [ ] Multi-DB distribué — une base par domaine/client, chaque DB sur le cluster
- [ ] Neo4j Fabric — requêtes fédérées cross-DB (ex: "cherche dans tous les graphs juridiques")
- [ ] Helm chart Neo4j pour déploiement K8s standardisé

## Serving de graphes distribués
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

## Inference GNN on-demand
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

## Visualisation interactive de graphes
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

## Graph on-demand — Création dynamique de graphes
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
