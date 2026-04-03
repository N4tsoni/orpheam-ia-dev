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

## Installer orpheam-libs en mode éditable (dev)

En dev, ne pas attendre de publier sur Nexus à chaque changement. Installer la lib locale en mode éditable :

```bash
cd ia/workers
.venv/bin/pip install -e ../../orpheam-libs/
```

Le `-e` (editable) crée un lien symbolique — chaque modif dans `orpheam-libs/` est immédiatement disponible sans réinstaller ni publier. Publier sur Nexus seulement une fois les changements validés.

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