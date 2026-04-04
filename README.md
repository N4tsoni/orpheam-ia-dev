# Orpheam IA Dev

Workspace de développement pour la stack IA d'Orpheam. Regroupe les deux submodules pour itérer simultanément sur la lib et les workers.

## Structure

```
Orpheam-IA-Dev/
    orpheam-libs/    ← Framework Python (orpheam package) — submodule
    ia/              ← Workers Celery + infra Docker — submodule
        workers/     ← Code source, tests, docker-compose
    docs/            ← Documentation Obsidian (guides, recaps, TODOs)
```

- **orpheam-libs** — Lib partagée : agents LangGraph, LLM routing (LiteLLM), Neo4j graph client (Graphiti), parsers, search config, Redis memory, MCP tools.
- **ia/workers** — Workers de production : Celery tasks (KG pipeline, embeddings, voice), RabbitMQ consumer, docker-compose.

## Principe

> Modifier la **lib** (`orpheam-libs`) en priorité. Le code **prod** (`ia/workers`) ne contient que la glue (Celery tasks, env vars, job payloads).

La lib est installée en mode éditable dans le venv des workers pour itérer sans publier sur Nexus :

```bash
cd ia/workers
.venv/bin/pip install -e ../../orpheam-libs/
```

## Setup

```bash
# Cloner avec submodules
git clone --recurse-submodules git@github.com:Orpheam/Orpheam-IA-Dev.git
cd Orpheam-IA-Dev

# Installer les dépendances workers
cd ia/workers
poetry config virtualenvs.in-project true
poetry install
.venv/bin/pip install -e ../../orpheam-libs/

# Démarrer l'infra de dev (Neo4j + LiteLLM)
make up

# Configurer les clés API
cp .env.example .env
# → Remplir OPENROUTER_API_KEY dans .env
```

## Commandes (ia/workers)

```bash
make              # aide
make test         # tests unitaires
make test-cov     # tests + couverture
make lint         # ruff + black check
make format       # formater le code
make up           # démarrer Neo4j + LiteLLM
make down         # arrêter
make install-lib  # réinstaller orpheam-libs en éditable
```

## Méthodologie de développement

Chaque feature suit ce cycle :

1. **Coder dans la lib** (`orpheam-libs`) — logique métier réutilisable
2. **Mettre à jour les imports** dans `ia/workers` (re-exports depuis la lib)
3. **Lancer les tests unitaires** (`make test`) — vérifie que rien ne casse
4. **Tester en notebook** avec les vrais services (Neo4j, LiteLLM) — validation end-to-end
5. **Commit** sur les deux submodules
6. **Mettre à jour la TODO et les docs**

## Tests

Deux types de tests :

- **pytest** (`make test`) — Tests unitaires rapides, sans infra, graphiti mocké. Valide que le code ne casse pas après un refactor.
- **Notebooks Jupyter** (`tests/notebooks/`) — Tests d'intégration interactifs avec les vrais services (Neo4j, LiteLLM, Graphiti). Nécessite `make up`.

| Notebook | Contenu |
|----------|---------|
| `01_litellm_proxy_test` | Validation proxy LiteLLM (health, LLM, embeddings, OrpheamRouter) |
| `02_kg_pipeline_test` | Pipeline KG complet (parse → ingest → report → inspection Neo4j) |
| `03_graphiti_litellm_test` | Graphiti via LiteLLM (indices, ingestion, recherche, OrpheamSearchConfig) |
| `04_orpheam_lib_stack_test` | Test complet de la stack lib (parsers, graphiti, search config, inspection) |

## Conventions de commit

Format : `type: description courte`

| Préfixe | Usage |
|---------|-------|
| `feat:` | Nouvelle fonctionnalité |
| `fix:` | Correction de bug |
| `refactor:` | Restructuration sans changement de comportement |
| `docs:` | Documentation uniquement |
| `test:` | Ajout/modification de tests |
| `chore:` | Maintenance (gitignore, deps, config) |

Exemples :
```
feat: add OrpheamSearchConfig with prebuilt search recipes
fix: proxy-compatible Graphiti client for LiteLLM (/v1/responses override)
refactor: migrate parsers from workers to orpheam-libs
docs: enrich graphiti notebook with explanatory markdown
chore: remove tracked __pycache__ files
```

## Architecture LLM

Tous les appels LLM/embeddings passent par un proxy LiteLLM centralisé :

```
Application / Notebook
    → orpheam-libs (OrpheamGraphitiClient, OrpheamRouter)
        → LiteLLM Proxy (http://litellm:4000/v1)
            → OpenRouter
                → Provider (Google Gemini, OpenAI, Anthropic)
```

Config des modèles dans `ia/workers/litellm_config.yaml`.

## Roadmap

Voir `docs/TODO - Sprint IA.md` pour le détail.

| Sprint | Statut |
|--------|--------|
| 1 — Intégration LiteLLM | ✅ Done |
| 2 — Pipeline KG (parsers, Graphiti, search) | 🔄 En cours |
| 3 — Graph Data Science (Neo4j GDS) | À venir |
| 4 — Refonte architecture | À évaluer |
