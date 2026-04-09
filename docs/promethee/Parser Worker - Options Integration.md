# Parser Worker — Options d'intégration

Deux options pour déclencher le parsing après upload des fichiers sur MinIO (S3).

> **Stockage** : MinIO (S3 auto-hébergé) remplace le FTP. SDK compatible S3 côté Laravel (PHP) et Python.

---

## Stockage — FTP vs MinIO (S3)

MinIO est un serveur de stockage objet auto-hébergé, compatible API S3 (Amazon). Il remplace le FTP.

|                        | FTP                               | MinIO (S3)                               |
| ---------------------- | --------------------------------- | ---------------------------------------- |
| Protocole              | FTP/SFTP (port 21/22)             | HTTP/HTTPS (port 9000)                   |
| Accès fichier          | chemin filesystem, volume partagé | API HTTP, presigned URLs                 |
| Docker                 | vsftpd ou similaire               | `minio/minio` (image officielle)         |
| Laravel config         | `Storage::disk('ftp')`            | `Storage::disk('s3')` (endpoint → MinIO) |
| Python accès           | `ftplib` (stdlib, basique)        | `boto3` ou `minio-py` (riche)            |
| Webhook / notification | non (polling uniquement)          | oui (notification native sur upload)     |
| Interface web          | non                               | oui (console MinIO, port 9001)           |
| Volume Docker partagé  | nécessaire entre services         | non — accès via API HTTP                 |
| Scaling                | un seul serveur                   | réplication, multi-node possible         |
| Auth                   | login/password                    | access key + secret key (standard AWS)   |

### Pourquoi MinIO pour nous

- **Pas de volume partagé** — le parse-worker Python télécharge via l'API S3, pas besoin de monter le même filesystem que Laravel
- **SDK standard** — même API que AWS S3, compatible partout (PHP, Python, Go...)
- **Webhook dispo** — si on choisit l'option B, c'est natif
- **Console web** — debug facile, voir les fichiers uploadés
- **Code existant** — file2md utilise déjà MinIO

---

## Option A — Job Laravel (upload + publish RabbitMQ)

Le job Laravel fait les deux : upload sur MinIO puis publie le job de parsing sur RabbitMQ.

```
User upload 10 fichiers → Laravel dispatch 10 jobs
  Job 1 : putObject() ✓ → publish RabbitMQ {path, user_id, project_id} → parse-worker
  Job 2 : putObject() ✓ → publish RabbitMQ {path, user_id, project_id} → parse-worker
  Job 3 : putObject() ✓ → publish RabbitMQ {path, user_id, project_id} → parse-worker
```

### Flux

```
┌──────────────────────────────────────┐
│         Job Laravel (async)          │
│                                      │
│  1. putObject() ──────▶ MinIO (S3)   │
│          ✓ retour = fichier prêt     │
│  2. publish() ──────▶ RabbitMQ       │
│                          │           │
└──────────────────────────────────────┘
                           │
                           ▼
                    ┌────────────┐     ┌────────────┐
                    │parse-worker│────▶│ kg-pipeline │
                    │  (Celery)  │     │ Pipeline V3 │
                    └────────────┘     └────────────┘
```

### Avantages
- **Contexte métier complet** — user_id, project_id, paramètres directement dans le message RabbitMQ
- **Stack unifiée** — tout dans RabbitMQ/Celery (monitoring Flower, retry, dead letter queue)
- **Pas d'infra supplémentaire** — pas de FastAPI, pas de config webhook MinIO
- **Retry natif** — Celery gère les retries, backoff, erreurs
- **Garantie fichier prêt** — putObject() bloquant, quand il retourne le fichier est sur MinIO

### Inconvénients
- **Couplé à Laravel** — seul Laravel peut déclencher un parsing
- **Deux responsabilités dans un job** — upload + publish, si crash entre les deux le fichier est sur MinIO mais le parsing jamais lancé
- **Laravel doit connaître RabbitMQ IA** — le job PHP publie directement sur la queue des workers Python

---

## Option B — Webhook MinIO → parse-worker

MinIO notifie directement le parse-worker quand un fichier arrive. Laravel n'intervient plus après l'upload.

```
User upload 10 fichiers → Laravel dispatch 10 jobs
  Job 1 : putObject() ✓ → done (Laravel a fini)
  Job 2 : putObject() ✓ → done
  Job 3 : putObject() ✓ → done

MinIO détecte les fichiers :
  webhook fichier 1 → parse-worker → parse
  webhook fichier 2 → parse-worker → parse
  webhook fichier 3 → parse-worker → parse
```

### Flux

```
┌─────────────────────────┐
│   Job Laravel (async)   │
│                         │
│  putObject() ──▶ MinIO  │
│       done              │
└─────────────────────────┘
                   │
                   │ webhook (POST automatique)
                   ▼
            ┌────────────┐     ┌────────────┐
            │parse-worker│────▶│ kg-pipeline │
            │ (FastAPI)  │     │ Pipeline V3 │
            └────────────┘     └────────────┘
```

### Avantages
- **Découplé** — Laravel ne sait rien du parsing, il upload et c'est fini
- **Garantie native MinIO** — le webhook ne se déclenche qu'après upload complet
- **Extensible** — tout upload sur MinIO (même hors Laravel) déclenche le parsing
- **Séparation des responsabilités** — Laravel = upload, MinIO = notification, Worker = parsing

### Inconvénients
- **Pas de contexte métier** — le webhook donne le nom du fichier, pas user_id/project_id → il faut soit une convention de path (`bucket/user_42/project_15/doc.pdf`), soit un lookup Redis/DB
- **FastAPI requis** — serveur HTTP pour recevoir le webhook, à maintenir et monitorer
- **Fiabilité** — si FastAPI est down, le webhook est perdu (MinIO retry limité)
- **Deux systèmes de jobs** — Celery pour le KG + FastAPI pour le parsing → monitoring éclaté

---

## Comparatif

| Critère | A — Job Laravel + RabbitMQ | B — Webhook MinIO |
|---|:---:|:---:|
| Qui déclenche le parsing | Laravel (publish RabbitMQ) | MinIO (webhook HTTP) |
| Contexte métier (user, project) | dans le message | convention path ou lookup |
| Fiabilité | RabbitMQ persiste | webhook peut être perdu |
| Monitoring | Flower unifié | Flower + FastAPI |
| Retry | Celery natif | à implémenter |
| Découplage Laravel ↔ IA | faible (Laravel publie sur queue IA) | fort (Laravel ne sait rien) |
| Infra supplémentaire | aucune | FastAPI |
| Upload hors-Laravel | non supporté | supporté nativement |
| Complexité | faible | moyenne |

---

## Recommandation

**Option A** — si Laravel est la seule source de fichiers. Plus simple, fiable, contexte métier inclus. Le "retour par Laravel" n'est pas un vrai aller-retour réseau — c'est juste deux appels séquentiels dans le même job PHP.

**Option B** — si on veut un découplage fort entre Laravel et l'IA, ou si d'autres sources que Laravel vont uploader des fichiers sur MinIO.

---

## Décision prise

**Stockage : MinIO (S3)** — remplace le FTP.
- Scalabilité horizontale (ajout de nœuds, erasure coding) — critique car le parsing de documents est notre cœur de métier, le volume va croître rapidement
- API standard S3 — migration cloud transparente si besoin (changement d'endpoint, code identique)
- Console web pour debug
- Code file2md déjà compatible

**Déclenchement : Option A — Job Laravel + RabbitMQ**
- Stack unifiée Celery/RabbitMQ, monitoring Flower, retry natif
- Contexte métier complet dans le message (user_id, project_id, paramètres)
- MinIO reste disponible pour ajouter un webhook plus tard si d'autres sources que Laravel doivent déclencher du parsing
