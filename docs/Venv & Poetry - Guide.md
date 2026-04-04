# Venv & Poetry - Guide

## Configuration

Poetry crée les venvs dans `~/.cache/pypoetry/virtualenvs/` par défaut. Pour les avoir localement dans chaque projet :

```bash
poetry config virtualenvs.in-project true
```

## Commandes utiles

```bash
# Créer/installer un venv
poetry install --no-root       # install deps sans le projet lui-même

# Voir le venv actif
poetry env info --path          # chemin complet du venv
poetry env list --full-path     # lister tous les venvs du projet

# Supprimer un venv
poetry env remove --all         # supprime tous les venvs du projet
rm -rf .venv                    # si venv local

# Vérifier qu'un module est installé
.venv/bin/python -c "import httpx; print(httpx.__version__)"
```

## Installer un package local en mode éditable (dev)

En dev, on peut installer un package Python local directement dans le venv Poetry **sans passer par Nexus/PyPI**. Utile pour itérer rapidement sur une lib.

```bash
cd ia/workers
.venv/bin/pip install -e ../../orpheam-libs/
```

- **`-e` (editable)** : crée un lien symbolique vers le code source. Chaque modif dans `orpheam-libs/` est immédiatement disponible sans réinstaller.
- **Sans `-e`** : `.venv/bin/pip install ../../orpheam-libs/` copie le package dans le venv (snapshot figé, il faut réinstaller à chaque changement).
- **Publier sur Nexus** seulement une fois les changements validés et testés.

Cela fonctionne pour n'importe quel package Python local avec un `pyproject.toml` ou `setup.py` :

```bash
# Installer un package local quelconque
.venv/bin/pip install -e /chemin/vers/mon-package/

# Vérifier qu'il est installé
.venv/bin/pip show mon-package
```

**Important** : `pip install -e` est indépendant de Poetry. Poetry gère le `pyproject.toml` et le lock, mais `pip` dans le `.venv` peut installer des packages supplémentaires. Ces packages ne seront pas dans `poetry.lock` — c'est voulu en dev.

## VS Code / Jupyter

- Le kernel Jupyter a besoin de `ipykernel` dans le venv :
  ```bash
  .venv/bin/pip install ipykernel
  ```
- Pour sélectionner l'interpréteur : **Ctrl+Shift+P** → "Python: Select Interpreter" → entrer le path `.venv/bin/python`
- Chaque dossier du workspace peut avoir son propre interpréteur

## Pièges courants

- `poetry install` sans `--no-root` échoue si le README n'existe pas → utiliser `--no-root`
- Changer `virtualenvs.in-project` après un `poetry install` ne migre pas l'ancien venv → supprimer l'ancien manuellement
- Un `.venv` vide prend priorité sur le venv dans le cache → le supprimer si les dépendances manquent
Pa