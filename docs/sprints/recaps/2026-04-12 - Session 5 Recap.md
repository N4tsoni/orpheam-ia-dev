# Daily — Session 5 Recap (12 avril 2026)

## Objectif

Restructurer la documentation Obsidian en sous-dossiers thématiques, éclater la TODO monolithique en 6 phases, enrichir le README.

## Ce qui a été fait

### 1. Restructuration docs Obsidian — sous-dossiers thématiques

**Constat** : tous les fichiers de docs étaient à plat dans chaque dossier (`promethee/`, `voice/`, `recherche/`). Difficile de s'y retrouver à mesure que la doc grossit.

**Restructuration** :

```
promethee/
  ├── graphe/              # Graphiti.md, Neo4j & Cypher.md, Entity Linking.md
  ├── parsing/             # Parser Worker - Options Integration.md
  ├── evaluation/          # Evaluation Pipeline KG - Guide.md
  ├── pipeline-v3/         # (inchangé)
  ├── Stack Technique.md   # (renommé, majuscule)
  └── README.md            # Liens mis à jour vers sous-dossiers

voice/
  ├── stt/                 # Whisper.md, Groq STT.md, Stack Technique.md, README.md
  ├── tts/                 # edge-tts.md, Stack Technique.md, README.md
  └── README.md            # Liens mis à jour

recherche/ → R&D/          # Renommage dossier, même contenu (concepts, papers, projets)
```

### 2. TODOs — éclatement en 6 phases

**Avant** : un seul fichier `TODO - Sprint IA.md` monolithique avec tout le backlog.

**Après** : un fichier par phase + un index.

| Fichier | Phase |
|---------|-------|
| `TODO-Phase1-Promethee.md` | Pipeline KG (en finalisation) |
| `TODO-Phase2-Apollon-Muses-Pneuma.md` | Orchestrateur multi-agents |
| `TODO-Phase3-Infra-K8s.md` | Infrastructure Kubernetes |
| `TODO-Phase4-Graph-Data-Science.md` | Neo4j GDS, algorithmes de graphe |
| `TODO-Phase5-GNN-Inference.md` | GNN & Inférence GPU |
| `TODO-Phase6-Quantum.md` | Quantum |

`TODO - Sprint IA.md` devient un simple index avec un tableau qui pointe vers chaque fichier.

### 3. Recaps de sessions — déplacés

Sessions 2, 3, 4 et le recap LiteLLM/Graphiti déplacés dans `docs/sprints/recaps/`.

### 4. README enrichi

- Section "A quoi sert ce repo" avec schéma du flux
- Process de dev numéroté (lib → workers → test → notebook → commit)
- Tableaux commandes dev + prod
- Section architecture LLM
- Roadmap 6 phases
- Arborescence docs

### 5. INDEX.md — liens corrigés

Mise à jour des liens après la restructuration (nouveaux chemins vers sous-dossiers).

### 6. Analyse RAM parser (non retenu)

Investigation du coût RAM du parser Marker. Le gros morceau est `create_model_dict()` (modèles PyTorch ~1.35 GB) + `MarkerPool` (N instances PdfConverter). Un notebook de profiling a été envisagé mais pas retenu — un simple script Python suffirait si besoin.

## Statut

| Composant | Statut |
|-----------|--------|
| Restructuration docs Obsidian | Done |
| TODOs éclatés en 6 phases | Done |
| Recaps déplacés dans recaps/ | Done |
| README enrichi | Done |
| INDEX.md liens corrigés | Done |
| Profiling RAM parser | Non retenu (pas besoin de notebook) |

## Prochaines étapes

- **Build parse-worker Docker** : `make build-worker` — premier test E2E Docker
- **Notebook 07 E2E** : test complet parse-worker Docker → MinIO → KG
- **Gold standard** : commencer annotation des documents de test
- **Phase 2** : commencer l'architecture Apollon / Muses / Pneuma
