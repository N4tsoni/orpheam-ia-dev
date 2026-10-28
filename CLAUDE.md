# CLAUDE.md

Dépôt parent d'un seul sous-module : **`ia/`**, les workers IA d'Orpheam. Reconstruction from scratch sur la branche `mvp`, en s'inspirant d'`origin/env/dev` sans la recopier.

## `ia/`

- **Code pur uniquement** : ni docs, ni notebooks, ni labs.
- Projet Poetry à la racine de `ia/`, package `orpheam_workers/`.
- `cd ia && make install` puis `make check` (ruff + mypy).

## Stack

`Laravel ─Kafka orpheam.ia.jobs─▶ dispatcher ─Celery (broker Redis)─▶ worker du pôle ─Kafka orpheam.ia.results─▶ Laravel`

- Graphe : **DozerDB** uniquement.
- Pas de lib `orpheam`, pas d'Umbreon, pas de RabbitMQ.

## Règles

- Code en anglais, docstrings et commentaires en français.
- Règles d'archi et décisions : `/home/n4/Bureau/N4_Brain/2-Projets/Orpheam/MVP/Prométhée/Todos/Todo Architecture Workers.md`.
- On avance fichier par fichier, et chaque choix est validé avant d'être codé.
