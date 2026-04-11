# Daily — Session 4 Recap (11 avril 2026)

## Objectif

Tester le parse-worker de bout en bout (Docker + Celery + MinIO), configurer le choix du parser, structurer Obsidian pour la recherche GDS.

## Ce qui a été fait

### 1. Notebook 06 — Section Parser + "METTEZ ICI LE PATH"

**Constat** : Le notebook testait les composants internes (complexity, enhance_markitdown) mais pas le vrai flux du parse-worker.

**Refonte** :
- Supprime les cellules de test de fonctions internes isolees
- Ajoute une section **"METTEZ ICI LE PATH"** : l'utilisateur configure `FILE_PATH` (fichier/dossier, local ou `s3://`) et le notebook parse tout automatiquement
- 4 modes : fichier MinIO, fichier local, dossier MinIO (prefix), dossier local
- Affichage detaille entree/sortie : taille, pages, apercu texte, Markdown, headings, temps

**Fix** : `TypeError: unsupported format string passed to PosixPath.__format__` — conversion `str(rel)` dans les f-strings.

### 2. Notebook 07 — Pipeline E2E (Laravel → Worker → MinIO → KG)

**Cree** : `tests/notebooks/07_e2e_pipeline_test.ipynb`

Test d'integration de bout en bout simulant le flux reel de production :

```
Laravel (simule)          RabbitMQ           parse-worker (Docker)
    |                        |                      |
    +-- JobPayload JSON ---->+----> queue parsing -->+
                             |                      |
                             |    1. Download MinIO  |
                             |    2. parse_document()|
                             |    3. Upload MinIO    |
                             |                      |
    +<-- JobResult JSON -----+<---- rpc:// ---------+
    |
    +-- Verification MinIO (bucket parsed/)
    +-- Pipeline KG Graphiti (optionnel)
```

| Etape | Description |
|-------|-------------|
| 0. Health Check | Teste tous les services (MinIO, RabbitMQ, Neo4j, LiteLLM, parse-worker) |
| 1. Upload MinIO | Simule l'upload Laravel sur MinIO |
| 2. JobPayload | Construit le JSON exactement comme Laravel (avec choix du parser) |
| 3. Celery send_task | Envoie au parse-worker Docker via RabbitMQ |
| 4. JobResult | Analyse le retour du worker (contrat Laravel) |
| 5. Verification | Download + inspection du Markdown depuis MinIO |
| 6. Pipeline KG | Graphiti extraction (optionnel, si Neo4j + LiteLLM up) |
| 7. Recap | Tableau de bord final |

### 3. Choix du parser configurable

**Modifie** :
- `orpheam_workers/infra/dto/payload.py` — champ `parser` dans `ParsingPayload`
- `orpheam_workers/parsing/pipeline.py` — `parse_document()` accepte le parametre `parser`, ajout `_pipeline_markitdown_file()`
- `orpheam_workers/parsing/tasks.py` — passe `parser` a `parse_document()`

| Parser | Description | Usage |
|--------|-------------|-------|
| `auto` | Hybride MarkItDown (trivial) + Marker (complexe) | Defaut, meilleur ratio qualite/perf |
| `marker` | Marker uniquement (OCR + vision) | PDFs scannes, images, layouts complexes |
| `markitdown` | MarkItDown uniquement (pdfminer) | PDFs textuels, plus rapide, plus leger |
| `sanitize` | Nettoyage seul | MD/TXT deja structures |

**Pret pour Kubernetes** : le champ `parser` dans le payload permet de router vers des workers specialises (pool Marker GPU vs pool MarkItDown CPU).

### 4. Docker compose test — RabbitMQ + parse-worker

**Modifie** : `docker-compose.test.yml`

Ajout de 2 services :
- **rabbitmq-test** — broker AMQP, management UI sur :15672 (orpheam / orpheam2024)
- **parse-worker-test** — worker Celery avec Marker, queue `parsing`, config legere (1 worker, 1 Marker)

**Bind mount** : `./data:/import:ro` — fichiers de test accessibles dans le container MinIO.

**Fix Dockerfile** : `libgl1-mesa-glx` → `libgl1` (renomme dans Debian Trixie).

### 5. Makefile — Nouvelles commandes

| Commande | Description |
|----------|-------------|
| `make up` | Demarrer toute l'infra de dev |
| `make up-worker` | Infra + parse-worker (build auto) |
| `make up-infra` | Infra seule sans workers |
| `make build-worker` | Build l'image parse-worker |
| `make logs-worker` | Logs du parse-worker |
| `make status` | Etat containers + RAM/CPU |

### 6. Structure Obsidian — Recherche GDS

**Dossier** : `docs/recherche/`

```
recherche/
  README.md             — Conventions (tags, vocabulaire normalise, workflow Zotero)
  papers/
    _TEMPLATE Paper.md  — Frontmatter YAML (titre, auteurs, annee, venue, DOI, zotero)
  concepts/
    _TEMPLATE Concept.md
    Knowledge Graph.md   — Definition + liens projets + algos
    Centrality.md        — PageRank, Betweenness, Degree, Closeness, Eigenvector
    Community Detection.md — Louvain, Leiden, Label Propagation
  auteurs/
    _TEMPLATE Auteur.md
  projets/
    _TEMPLATE Projet.md
    Neo4j GDS.md         — Algorithmes, modes execution, integration Orpheam
    Graphiti.md          — Architecture, temporalite, integration pipeline
```

**Impact sur la qualite du KG** :
- Vocabulaire normalise → evite les doublons d'entites
- Liens `[[concept]]` Obsidian → relations explicites captees par le LLM
- Separation par type → `group_id` distincts pour Graphiti
- Templates YAML → metadonnees structurees exploitables par le pipeline

### 7. Verification graphiti-core 0.3.21

**Verification dans le code source installe** (`openai_client.py:86`) :
- `OpenAIClient` utilise deja `chat.completions.create()` avec JSON mode nativement
- Compatible avec LiteLLM sans proxy → suppression `_ProxyCompatibleOpenAIClient` confirmee
- `LLMConfig` : `api_key`, `model`, `base_url`, `temperature`, `max_tokens` uniquement

### 8. Divers

- **Gitignore** : `ia/workers/.gitignore` cree, `poetry.lock` retire du tracking
- **INDEX.md** : section Recherche ajoutee, liens sprints corriges
- **TODO** : prochaines etapes mises a jour
- **Zotero 9.0.0** : installe via APT pour les notes de lecture papers

## Tests effectues

| Test | Resultat |
|------|----------|
| Marker — premier lancement | OK (telechargement modeles ~1.35 GB, cache dans `~/.cache/datalab/`) |
| parse_document() sur PDF | OK — 327 chars, 21 lignes, 0.02s |
| parse_document() sur TXT | OK |
| parse_document() sur DOCX | ERREUR — `weasyprint` manquant en local (OK en Docker) |
| Upload fichiers MinIO | OK — bucket `documents/test/` |
| Docker stats | MinIO 102MB, LiteLLM 56MB, Neo4j 762MB |
| RabbitMQ | Service ajoute au compose, pas encore teste (build worker en cours) |

## Commits

**ia/workers** :
```
feat: choix du parser + notebook 07 E2E + RabbitMQ/parse-worker docker

- ParsingPayload.parser : auto, marker, markitdown, sanitize
- parse_document() accepte le choix du parser, ajout pipeline markitdown-only
- Notebook 07 : test E2E Laravel → Celery → parse-worker Docker → MinIO → KG
- Notebook 06 : section "METTEZ ICI LE PATH"
- docker-compose.test.yml : ajout RabbitMQ + parse-worker standalone
- Makefile : up-worker, up-infra, logs-worker, status, build-worker
- Dockerfile.parsing : fix libgl1 pour Debian Trixie
- .gitignore : poetry.lock retiré
```

**orpheam-libs** :
```
fix: suppression ProxyCompatibleOpenAIClient (graphiti-core 0.3.21 utilise déjà chat.completions nativement)
```

**parent repo** :
```
docs: structure Obsidian recherche GDS + sprint 4 recap + TODO
```

## Statut

| Composant | Statut |
|-----------|--------|
| Notebook 06 — parser tests + METTEZ ICI LE PATH | Done |
| Notebook 07 — E2E pipeline | Done |
| Choix du parser configurable | Done |
| Docker compose — RabbitMQ + parse-worker | Done (build en cours) |
| Makefile — commandes dev | Done |
| Structure Obsidian recherche | Done |
| graphiti-core 0.3.21 compat | Done (verifie) |
| Bind mount MinIO | Done |
| Gitignore ia/workers | Done |
| Zotero | Installe |

## Prochaines etapes

- **Build parse-worker** : `make build-worker` (premier build long, ensuite cache Docker)
- **Tester E2E** : notebook 07 complet (parse-worker Docker → MinIO → KG)
- **Mise en prod** : Laravel ↔ MinIO ↔ RabbitMQ (Sprint 4.1)
- **Benchmark** : notebook dedie evaluation, gold standard (Sprint 4.3)
- **Obsidian** : fiches de lecture papers GDS + plugin Zotero Integration
- **Kubernetes** : routing workers par type de parser (GPU Marker vs CPU MarkItDown)
