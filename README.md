# Orpheam IA Dev

Workspace de développement pour la stack IA d'Orpheam. Regroupe deux submodules git pour itérer simultanément sur la lib et les workers.

## A quoi sert ce repo

Orpheam IA est une plateforme distribuée qui transforme des **documents bruts** en **graphes de connaissances** (Knowledge Graphs) exploitables par des agents LLM. Laravel envoie des jobs via RabbitMQ, les workers Python les traitent (parsing, extraction KG, embeddings, voice) et renvoient les résultats.

```
Laravel → RabbitMQ → Consumer → Celery Workers → Neo4j / MinIO → RabbitMQ → Laravel
```

## Structure

```
orpheam-ia-dev/
├── orpheam-libs/    # Framework Python (package orpheam) — submodule
├── ia/
│   └── workers/     # Workers Celery + Docker — submodule
└── docs/            # Documentation Obsidian
```

- **orpheam-libs** — Lib partagée : agents LangGraph, LLM routing (LiteLLM), Neo4j client (Graphiti), parsers, MCP tools, Redis memory.
- **ia/workers** — Code prod : Celery tasks, RabbitMQ consumer, docker-compose. Glue uniquement — la logique est dans la lib.

## Process de dev

> Modifier la **lib** en priorité. Les workers ne contiennent que la glue (tasks Celery, env vars, DTOs).

```
1. Coder dans orpheam-libs/         # logique metier reutilisable
2. Mettre a jour ia/workers/        # imports, tasks Celery si besoin
3. make test                        # tests unitaires (pas d'infra)
4. make up && make notebook         # notebook Jupyter avec vrais services
5. cd orpheam-libs && git commit    # commit submodule lib
   cd ia && git commit              # commit submodule workers
6. cd .. && git add ia orpheam-libs # commit repo principal (pointeurs)
   git commit -m "Update SubModule"
```

## Setup

```bash
# Cloner avec submodules
git clone --recurse-submodules git@github.com:Orpheam/Orpheam-IA-Dev.git
cd Orpheam-IA-Dev

# Installer les deps
cd ia/workers
poetry config virtualenvs.in-project true
poetry install
make install-lib          # orpheam-libs en mode editable

# Config
cp .env.example .env      # remplir OPENROUTER_API_KEY

# Demarrer l'infra dev
make up
```

## Commandes dev (ia/workers)

| Commande | Description |
|----------|-------------|
| `make install` | Installer les deps principales |
| `make install-lib` | Installer orpheam-libs en editable |
| `make install-all` | Toutes les deps (parsing + voice + dev) |
| `make test` | Tests unitaires |
| `make test-cov` | Tests + couverture |
| `make check` | Lint + typecheck + tests |
| `make lint` | Ruff + Black check |
| `make format` | Formater le code |
| `make up` | Demarrer l'infra dev (Neo4j + LiteLLM + MinIO + RabbitMQ) |
| `make up-worker` | Infra + parse-worker Docker |
| `make down` | Arreter l'infra |
| `make status` | Etat des containers + RAM/CPU |
| `make logs` | Logs de l'infra |
| `make notebook` | Jupyter Lab (session tmux) |
| `make clean` | Nettoyer les caches Python |

## Commandes prod (ia/workers)

| Commande | Description |
|----------|-------------|
| `make up-prod` | Demarrer les workers prod (necessite orpheam-network) |
| `make down-prod` | Arreter les workers prod |
| `make build-worker` | Build l'image Docker worker |

> **Prod** requiert le réseau Docker externe `orpheam-network` (créé par le repo Laravel principal qui fait tourner Neo4j, RabbitMQ, Postgres, etc.).

## Tests

- **pytest** (`make test`) — Tests unitaires rapides, sans infra, mocks.
- **Notebooks** (`make notebook`) — Tests d'intégration avec les vrais services. Necessite `make up`.

## Architecture LLM

```
Code / Notebook
  → orpheam-libs (OrpheamRouter, OrpheamGraphitiClient)
    → LiteLLM Proxy (http://litellm:4000)
      → OpenRouter
        → Gemini, Claude, GPT...
```

## Roadmap

Voir `docs/TODO - Sprint IA.md` — index des 6 phases :

| Phase | Sujet | Statut |
|-------|-------|--------|
| 1 | Promethee — Pipeline KG | En finalisation |
| 2 | Apollon, Muses & Pneuma — Orchestrateur multi-agents | A venir |
| 3 | Infrastructure Kubernetes | A venir |
| 4 | Graph Data Science | A venir |
| 5 | GNN & Inference GPU | A venir |
| 6 | Quantum | A venir |

## Documentation

```
docs/
├── promethee/       # Pipeline KG (graphe, parsing, pipeline-v3, evaluation)
├── apollon/         # Orchestrateur (a venir)
├── voice/           # STT & TTS (stt/, tts/)
├── infra/           # Docker, RabbitMQ, MinIO
├── recherche/       # Papers, notes
├── guides/          # Guides pratiques
├── sprints/         # TODOs par phase + recaps de sessions
└── Plan.md          # Roadmap 6 phases
```
