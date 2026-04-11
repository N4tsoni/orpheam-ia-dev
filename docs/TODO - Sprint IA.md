# TODO - Sprint IA

> Organisation par **Phases**. Chaque phase a son propre fichier dans `docs/sprints/`.

---

## Fichiers TODO

| Phase | Fichier | Statut |
|-------|---------|--------|
| **Phase 1** — Promethee : Pipeline KG | [TODO-Phase1-Promethee.md](sprints/TODO-Phase1-Promethee.md) | ✅ En finalisation |
| **Phase 2** — Apollon, Muses & Pneuma | [TODO-Phase2-Apollon-Muses-Pneuma.md](sprints/TODO-Phase2-Apollon-Muses-Pneuma.md) | A venir |
| **Phase 3** — Infrastructure Kubernetes | [TODO-Phase3-Infra-K8s.md](sprints/TODO-Phase3-Infra-K8s.md) | A venir |
| **Phase 4** — Graph Data Science | [TODO-Phase4-Graph-Data-Science.md](sprints/TODO-Phase4-Graph-Data-Science.md) | A venir |
| **Phase 5** — GNN & Inférence GPU | [TODO-Phase5-GNN-Inference.md](sprints/TODO-Phase5-GNN-Inference.md) | A venir |
| **Phase 6** — Quantum | [TODO-Phase6-Quantum.md](sprints/TODO-Phase6-Quantum.md) | A venir |

## Recaps de sessions

Dans `docs/sprints/recaps/` (format `YYYY-MM-DD - Session N Recap.md`).

---

## Notes

- **Nommage** : Promethee (graphs), Apollon (orchestrateur), Muses (sub-agents), Pneuma (skills MCP)
- **Methodo** : coder dans la lib → mettre a jour workers → `make test` → notebook → commit → docs
- **Regle** : modifier la lib (`orpheam-libs`) en priorite, le code prod (`ia/workers`) seulement si specifique au domaine
- **Tests/Docs** : Jupyter notebooks dans le dossier test
- **LLM en dev** : switcher facilement de modele via OpenRouter + proxy LiteLLM
- **Pipeline V3** : developpe par Laz, integre dans la nouvelle archi — doc dans `docs/promethee/pipeline-v3/`
