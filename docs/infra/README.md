# Infra — Communication & Orchestration

La couche infra est le **pont entre Laravel et les workers Python**. Elle route les jobs, sérialise les données, et renvoie les résultats.

## Comment ca marche

```
Laravel → RabbitMQ → Consumer (pika) → Celery → Worker → Résultat → RabbitMQ → Laravel
```

## Docs du dossier

- [Stack technique](Stack%20technique.md) — Technos, queues, DTOs, fichiers clés
- [RabbitMQ & Celery](RabbitMQ%20&%20Celery.md) — Broker de messages et task queue
- [LiteLLM](LiteLLM.md) — Proxy LLM centralisé
- [Recap - Integration LiteLLM](Recap%20-%20Integration%20LiteLLM.md) — Journal technique Sprint 1
