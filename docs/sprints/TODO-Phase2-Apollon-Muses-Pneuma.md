# TODO Phase 2 — Apollon, Muses & Pneuma : Orchestrateur Multi-Agents

> Objectif : orchestrateur intelligent (Apollon) qui route vers des sub-agents configurables (Muses),
> équipés de skills MCP (Pneuma) et connectés à des modules de graphs (Promethee).
>
> **Architecture cible** :
> ```
> User → Apollon (orchestrateur)
>            ├── Muse A (LLM X + skills Pneuma [web, calc] + Graph module juridique)
>            ├── Muse B (LLM Y + skills Pneuma [code, search] + Graph module technique)
>            └── Muse C (LLM Z + skills Pneuma [vision] + Graph module médical)
> ```
>
> Une **Muse** = choix LLM + ensemble de skills MCP (Pneuma) + modules de graphs (petits ou gros, par catégorie).
> La gestion des catégories de Muses se fait côté web (Laravel).

---

## 2.1 Refonte Apollon — Orchestrateur Central
- [ ] Remplacer Apollon legacy (LangGraph monolithique) par `OrpheamSupervisor`
- [ ] Routage dynamique : analyse de la requête → sélection de la Muse appropriée
- [ ] Gestion de contexte : mémoire partagée entre Apollon et les Muses (Redis checkpointer)
- [ ] Multi-turn : conversation continue, Apollon maintient l'état global
- [ ] Fallback : si une Muse échoue, Apollon reroute vers une autre

## 2.2 Muses — Sub-Agents Configurables
- [ ] Interface `Muse` — agent configurable : LLM + skills + graphs
- [ ] Configuration dynamique : une Muse est définie par un JSON/config (pas de code)
- [ ] Registre de Muses — discovery et instanciation à la demande
- [ ] Chaque Muse connectée à un ou plusieurs modules de graphs Promethee
- [ ] A2A (Agent-to-Agent) — communication inter-Muses avec circuit breaker
- [ ] Lifecycle hooks : init, pre-execute, post-execute, cleanup

## 2.3 Pneuma — Skills MCP
- [ ] Registre MCP centralisé — skills disponibles pour toutes les Muses
- [ ] Skills de base : KG search, file parsing, web search, calculator
- [ ] Intercepteurs MCP : logging, rate limiting, auth, cache
- [ ] Skills composables : un skill peut appeler d'autres skills
- [ ] Hot-reload : ajouter/retirer des skills sans redémarrer
- [ ] SDK skill : template pour créer un nouveau skill MCP rapidement

## 2.4 Computer Vision
- [ ] Intégration vision pour documents complexes (layouts, diagrammes, schémas)
- [ ] OCR avancé : tableaux imbriqués, formules mathématiques, handwriting
- [ ] Extraction structurée depuis images → entités/relations KG
- [ ] Pipeline multimodal : texte + images + tableaux dans un document
- [ ] Benchmark vision models via OpenRouter (GPT-4V, Claude Vision, Gemini Pro Vision)

## 2.5 Optimisation LLM
- [ ] Routing intelligent : modèle léger (tâches simples) vs lourd (extraction complexe)
- [ ] Chaîne de fallback adaptive (coût vs qualité vs latence)
- [ ] Few-shot / prompts spécialisés par domaine
- [ ] Distillation : modèle local entraîné sur outputs des gros modèles

## 2.6 Intégration Laravel
- [ ] API REST/WebSocket pour créer/configurer des Muses depuis le web
- [ ] Dashboard : état des Muses, graphs connectés, skills actifs
- [ ] Multi-tenant : isolation par user/organisation
- [ ] Clés API users gérées côté Laravel

## 2.7 Mise en Prod & Kubernetes
- [ ] Docker images multi-stage optimisés (CPU vs GPU)
- [ ] Healthchecks — endpoints `/health` pour chaque worker
- [ ] Logs structurés JSON (Loki, ELK)
- [ ] CI/CD — GitHub Actions : lint + tests sur PR, build Docker sur merge
- [ ] Kubernetes — déploiement K8s (voir TODO-Phase3-Infra-K8s.md)
