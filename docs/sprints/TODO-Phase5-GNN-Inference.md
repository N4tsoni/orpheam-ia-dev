# TODO Phase 5 — GNN & Inférence GPU

> Objectif : Graph Neural Networks classiques pour enrichir les KG — node classification,
> link prediction, graph embeddings. Baseline avant le quantum.

---

## 5.1 GNN classiques — Graph Neural Networks
> PyTorch Geometric (PyG) pour du message passing sur GPU.

- [ ] POC GNN sur sous-graphe Neo4j (node classification, link prediction)
- [ ] Comparer GCN, GAT, GraphSAGE sur les KG Orpheam
- [ ] Benchmark vs algorithmes GDS natifs (qualité, vitesse)
- [ ] Évaluer si le SCT de Laz (LightweightGNN) suffit ou si PyG apporte un gain

## 5.2 Serving GNN on-demand
- [ ] **Serving GNN** — modèles PyG/ONNX derrière une API (Triton, TorchServe, ou FastAPI)
  - GPU pool partagé K8s (NVIDIA GPU Operator)
  - Auto-scaling : scale-to-zero quand pas de requêtes, scale-up à la demande
  - Batch inference : grouper les requêtes pour maximiser le throughput GPU
- [ ] **GNN as a Service** — les Muses appellent l'inference GNN via Pneuma (skill MCP)
  - Node classification on-demand (ex: "de quel type est cette entité ?")
  - Link prediction on-demand (ex: "quelles relations manquent ?")
  - Graph embedding on-demand (ex: "représente ce sous-graphe en vecteur")

## 5.3 Pipeline d'entraînement
- [ ] **Pipeline GNN** — entraînement périodique sur les KG mis à jour
  - CronJob K8s : ré-entraîner le GNN quand le graph évolue significativement
  - Model registry (MLflow ou simple MinIO) : versioning des modèles
  - A/B serving : comparer deux versions de modèle en production

## 5.4 Distillation & modèles locaux
- [ ] Distillation : modèle local entraîné sur outputs des gros modèles LLM
- [ ] Routing intelligent : modèle léger (tâches simples) vs lourd (extraction complexe)
- [ ] Chaîne de fallback adaptive (coût vs qualité vs latence)
- [ ] Few-shot / prompts spécialisés par domaine
