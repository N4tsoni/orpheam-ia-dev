# TODO Phase 6 — Quantum Graph Neural Networks

> Point de convergence KG + Quantum Computing.
> Les graphes s'encodent naturellement sur circuits quantiques (nœuds → qubits, arêtes → intrication).

---

## 6.1 Recherche
- [ ] Revue de littérature : QGNN, Quantum Walk, VQE, QAOA pour graphes
- [ ] Fiches Obsidian (`recherche/papers/`) : papiers clés
- [ ] Évaluer frameworks : PennyLane, Qiskit, Cirq, TensorFlow Quantum

## 6.2 Accès au calcul quantique

| Fournisseur            | Accès                                                            | Qubits                       | Coût estimé                |
| ---------------------- | ---------------------------------------------------------------- | ---------------------------- | -------------------------- |
| **IBM Quantum**        | Qiskit Runtime, plan gratuit (simulateurs) + pay-as-you-go (QPU) | 127-1121 (Eagle, Heron)      | ~1.60$/sec QPU             |
| **Amazon Braket**      | AWS, on-demand                                                   | IonQ (36), Rigetti (84), IQM | ~0.30$/task + shot pricing |
| **Google Quantum AI**  | Cirq, accès recherche                                            | Sycamore (72)                | Programme recherche        |
| **Azure Quantum**      | Credits gratuits ($500), IonQ/Quantinuum                         | Variable                     | Pay-per-job                |
| **IonQ**               | Direct API, Qiskit/Cirq/Braket                                   | Forte (36 qubits)            | $0.01/gate shot            |
| **Simulateurs locaux** | PennyLane `default.qubit`, Qiskit Aer                            | ~25-30 qubits (RAM)          | Gratuit                    |

> **Stratégie** : prototyper sur simulateur local (PennyLane), valider sur IBM Quantum (gratuit),
> scaler sur Amazon Braket ou Azure Quantum pour les vrais benchmarks.

## 6.3 Prototypage
- [ ] Simulateur local PennyLane (`default.qubit`, ~25 qubits)
- [ ] Sous-graphe test extrait de Neo4j (100-500 nœuds)
- [ ] POC détection de communautés — QAOA vs Louvain classique
- [ ] POC link prediction — QGNN variationnel vs GNN classique (PyG)
- [ ] Notebook `quantum_graph_poc.ipynb`

## 6.4 Cas d'usage
- [ ] Détection de communautés massives (QAOA / MaxCut, superposition 2^n partitions)
- [ ] Similarité structurelle (Quantum Walk, analogies inter-domaines)
- [ ] Requêtes multi-hop (Grover O(√N) vs O(N) classique)
- [ ] Quantum PageRank + VQE multi-objectifs
- [ ] Prédiction de liens (espace de Hilbert exponentiel)

## 6.5 Intégration Orpheam
- [ ] Module `orpheam/quantum/` dans orpheam-libs
- [ ] Interface unifiée : même API que GDS classique, backend quantum transparent
- [ ] Hybrid pipeline : pré-traitement classique → calcul quantique → post-traitement
- [ ] Routage auto : classique (petits graphes) vs quantum (graphes massifs)
- [ ] Accès cloud QPU (IBM Quantum Runtime, Amazon Braket)
