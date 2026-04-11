# Neo4j & Cypher

## C'est quoi ?

**Neo4j** est une base de données orientée graphe. Les données sont des **noeuds** reliés par des **relations**, pas des tables.

**Cypher** est son langage de requete. Il décrit des patterns visuels :

```cypher
(alice:Person)-[:WORKS_AT]->(orpheam:Company)
```

## Schéma Graphiti dans Neo4j

### Noeuds

| Label | Rôle | Propriétés |
|-------|------|------------|
| `Entity` | Entités extraites | `name`, `summary`, `group_id`, `embedding` |
| `Episodic` | Texte source ingéré | `name`, `content`, `reference_time` |
| `Community` | Clusters d'entités | `name`, `summary` |

### Relations

| Type | Entre | Propriétés |
|------|-------|------------|
| `RELATES_TO` | Entity → Entity | `fact`, `valid_at`, `expired_at`, `embedding` |
| `MENTIONS` | Episodic → Entity | Lie épisode → entités générées |
| `HAS_MEMBER` | Community → Entity | Appartenance communauté |

## Requetes courantes

### Explorer
```cypher
-- Tout voir (limité)
MATCH (n) RETURN n LIMIT 100

-- Compter par type
MATCH (n) RETURN labels(n), count(*)

-- Entités d'un groupe
MATCH (n:Entity) WHERE n.group_id = "mon-groupe" RETURN n.name, n.summary
```

### Entités & Relations
```cypher
-- Chercher une entité
MATCH (n:Entity) WHERE toLower(n.name) CONTAINS "alice" RETURN n

-- Faits valides
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
WHERE r.expired_at IS NULL
RETURN a.name, r.fact, b.name

-- Contexte d'une entité (1 saut)
MATCH (n:Entity {name: "Alice"})-[r]-(m) RETURN n, r, m

-- Entités les plus connectées
MATCH (n:Entity)-[r]-() RETURN n.name, count(r) AS deg ORDER BY deg DESC LIMIT 10
```

### Diagnostic
```cypher
-- Noeuds orphelins
MATCH (n:Entity) WHERE NOT (n)-[:RELATES_TO]-() RETURN count(n)

-- Densité (ratio edges/nodes, bon > 1.5)
MATCH (n:Entity) WITH count(n) AS nodes
MATCH ()-[r:RELATES_TO]->() WITH nodes, count(r) AS edges
RETURN edges, nodes, toFloat(edges)/nodes AS density

-- Doublons potentiels
MATCH (a:Entity), (b:Entity)
WHERE a <> b AND apoc.text.levenshteinSimilarity(a.name, b.name) > 0.85
RETURN a.name, b.name LIMIT 20
```

### Temporalité
```cypher
-- Faits valides à une date
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
WHERE r.valid_at <= datetime("2024-03-01T00:00:00Z")
  AND (r.expired_at IS NULL OR r.expired_at > datetime("2024-03-01T00:00:00Z"))
RETURN a.name, r.fact, b.name
```

## Tips Neo4j Browser

- Double-clic sur un noeud → voir les voisins
- `:schema` → afficher le schéma
- `Tab` → auto-completion
