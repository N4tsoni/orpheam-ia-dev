# Infra — Stack technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| RabbitMQ | AMQP 0.9.1 | Broker de messages |
| Celery | `celery ^5.3.0` | Task queue distribuée |
| pika | `pika` | Client AMQP (consumer standalone) |
| LiteLLM | Proxy HTTP port 4000 | Gateway LLM |
| Flower | `flower` | Monitoring Celery |
| Pydantic | `pydantic-settings` | Config + DTOs |

## Queues Celery

| Queue | Routing Key | Workers | Concurrency |
|-------|-------------|---------|-------------|
| `kg-pipeline` | `kg.#` | kg-worker | 2 |
| `embeddings` | `embed.#` | embedding-worker | 2 |
| `ml-inference` | `ml.#` | kg-worker | 2 |
| `voice` | `voice.#` | voice-worker | 1 |
| `default` | `default` | - | - |

## DTOs

### JobPayload (entrée depuis Laravel)
```json
{
    "job_id": "uuid",
    "type": "kg_extraction",
    "data": { },
    "user_id": "uuid",
    "graph_id": "uuid"
}
```

### JobResult (retour vers Laravel)
```json
{
    "job_id": "uuid",
    "status": "success",
    "result": { },
    "metrics": { "duration_seconds": 12.5 }
}
```

## Docker

| Fichier | Usage | Réseau |
|---------|-------|--------|
| `docker-compose.yml` | Prod | `orpheam-network` (externe) |
| `docker-compose.test.yml` | Dev/test standalone | Aucun |

```bash
docker compose -f docker-compose.test.yml up -d   # Dev
docker compose up -d                                # Prod
```

## Fichiers clés

```
ia/workers/orpheam_workers/infra/
├── celery_app.py     # Config Celery
├── consumer.py       # Consumer RabbitMQ standalone (pika)
├── dispatch.py       # Routage JobType → Celery task
├── amqp.py           # Publisher résultats
├── file_source.py    # Résolution fichiers (local, FTP, HTTP)
└── dto/
    ├── payload.py    # JobPayload
    └── result.py     # JobResult
```
