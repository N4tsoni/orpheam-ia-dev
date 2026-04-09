# RabbitMQ & Celery

## RabbitMQ — C'est quoi ?

[RabbitMQ](https://www.rabbitmq.com/) est un **broker de messages** (AMQP). Les applications s'envoient des messages via des queues sans se connaitre directement.

Dans notre cas : **Laravel publie des jobs** → RabbitMQ → **les workers Python les consomment**.

### Nos queues

| Queue | Direction | Contenu |
|-------|-----------|---------|
| `ia.kg-pipeline` | Laravel → Workers | Jobs extraction KG |
| `ia.embeddings` | Laravel → Workers | Jobs embeddings |
| `ia.voice` | Laravel → Workers | Jobs STT/TTS |
| `ia.ml-inference` | Laravel → Workers | Jobs assistant |
| `ia.results` | Workers → Laravel | Résultats des jobs |

### Exchange

`orpheam.ia.jobs` — exchange qui distribue les jobs vers les bonnes queues.

---

## Celery — C'est quoi ?

[Celery](https://docs.celeryq.dev/) est une **task queue distribuée** en Python. On définit des tâches (fonctions), Celery les exécute en arriere-plan sur des workers.

```python
@shared_task(bind=True, max_retries=3)
def process_kg_extraction(self, payload_json: str):
    # ... traitement ...
    return result
```

### Notre setup

- **Broker** : RabbitMQ (AMQP)
- **Result backend** : RPC (résultats via AMQP, pas Redis)
- **Sérialisation** : JSON uniquement
- **Prefetch** : 1 (un message a la fois par worker)

### Consumer standalone

On a un consumer **pika** séparé (`consumer.py`) qui écoute les queues Laravel et dispatche vers Celery. C'est un pont entre le format de messages Laravel et les tâches Celery.

```
RabbitMQ (queues ia.*) → consumer.py (pika) → celery_app.send_task()
```

Pourquoi pas Celery directement ? Parce que Laravel publie dans un format custom (JobPayload JSON), pas dans le format natif Celery.

### Retry

Les tâches ont des retries automatiques :
- `autoretry_for=(Exception,)` — retry sur toute erreur
- `max_retries` — limite (2-3 selon la tâche)
- `default_retry_delay` — délai entre retries (10s-120s)

### Monitoring

**Flower** : dashboard web pour voir les tâches en cours, les workers actifs, les erreurs.
