# Recap — Intégration LiteLLM (Sprint 1)

## Ce qui a été fait

- Service LiteLLM ajouté au `docker-compose.test.yml` (standalone, pas de réseau externe)
- `litellm_config.yaml` corrigé avec les bons model IDs OpenRouter
- `OrpheamRouter` (lib) adapté pour communiquer via le proxy LiteLLM
- Notebook de test `litellm_proxy_test.ipynb` créé et validé
- `.gitignore` ajouté au repo `ia` (manquait, des .venv et __pycache__ étaient committés)

## Erreurs rencontrées et solutions

### 1. `orpheam-network` not found
**Cause** : `docker-compose.yml` (prod) dépend d'un réseau Docker externe créé par le repo principal Orpheam.
**Solution** : Utiliser `docker-compose.test.yml` pour le dev/test, qui est standalone sans réseau externe.

### 2. Healthcheck LiteLLM `unhealthy`
**Cause** : `curl` n'est pas installé dans l'image LiteLLM.
**Solution** : Remplacer par `python -c "import urllib.request; urllib.request.urlopen(...)"`.

### 3. Healthcheck 401 Unauthorized
**Cause** : L'endpoint `/health` requiert l'authentification (master key).
**Solution** : Utiliser `/health/readiness` qui ne requiert pas d'auth.

### 4. `LLM Provider NOT provided` via OrpheamRouter
**Cause** : `litellm.completion()` appelé localement essaie de résoudre le provider depuis le nom du modèle (`google/gemini-2.5-flash`) au lieu de forward au proxy.
**Solution** : Ajouter `custom_llm_provider="openai"` dans les appels `completion()` pour traiter le proxy comme un endpoint OpenAI-compatible.

### 5. `Missing Anthropic API Key` sur les fallbacks
**Cause** : Les fallbacks sont exécutés par litellm **localement** (pas via le proxy), donc il essaie d'appeler Anthropic directement sans clé.
**Solution** : Supprimer les `fallbacks=` des appels `completion()`. Les fallbacks doivent être gérés côté proxy (dans `litellm_config.yaml`), pas côté client.

### 6. `No api key passed in` sur les appels au proxy
**Cause** : `OrpheamRouter` n'envoyait pas la master key LiteLLM.
**Solution** : Ajout du paramètre `api_key` sur `OrpheamRouter` et `_OrpheamChatModel`, passé dans `completion(api_key=...)`.

### 7. `Result.is_ok()` → `AttributeError`
**Cause** : L'API `Result` d'orpheam utilise `.ok` (propriété), pas `.is_ok()` (méthode). Le CLAUDE.md de la lib indiquait `is_ok()` à tort.
**Solution** : Remplacer `.is_ok()` par `.ok` dans tout le notebook.

### 8. Model IDs invalides sur OpenRouter
**Cause** : `google/gemini-2.5-flash-lite-preview` et `anthropic/claude-3.5-sonnet` ne sont pas des IDs valides sur OpenRouter.
**Solution** : Corriger dans `litellm_config.yaml` → `google/gemini-2.0-flash-lite-001` et `anthropic/claude-sonnet-4`.

### 9. `completion_cost()` unexpected keyword argument
**Cause** : L'API `litellm.completion_cost()` n'accepte pas `prompt_tokens`/`completion_tokens` directement.
**Solution** : Utiliser `litellm.cost_per_token()` à la place, avec préfixe `openrouter/` pour le cost mapping.

### 10. Venv vide / mauvais interpréteur
**Cause** : `poetry config virtualenvs.in-project true` crée un `.venv` local mais les dépendances restaient dans l'ancien venv (cache Poetry). Le nouveau `.venv` était vide.
**Solution** : Supprimer le `.venv` vide, `poetry install --no-root` pour recréer avec les deps. Installer `ipykernel` pour Jupyter. Installer orpheam-libs en mode éditable (`pip install -e`).

## Modifs dans orpheam-libs

**Fichier** : `orpheam/llm/router.py`
- `OrpheamRouter.__init__()` : nouveau param `api_key`
- `_OrpheamChatModel.__init__()` : nouveau param `api_key`
- `_OrpheamChatModel._call()` : ajout `custom_llm_provider="openai"`, `api_key=`, suppression `fallbacks=`
- `_OrpheamChatModel._stream_chunks()` : idem
- `OrpheamRouter.get_cost_estimate()` : `cost_per_token()` avec fallback `openrouter/` prefix

## Modifs dans ia/workers

- `docker-compose.test.yml` : ajout service `litellm`
- `docker-compose.yml` : fix healthcheck
- `litellm_config.yaml` : correction model IDs, ajout `health_check_ephem_key: false`
- `tests/notebooks/litellm_proxy_test.ipynb` : nouveau notebook de validation
- `.gitignore` : créé (manquait)
