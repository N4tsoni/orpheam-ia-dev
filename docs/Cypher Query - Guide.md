# Cypher Query — Guide & Exemples

## Les bases de Cypher

Cypher est le langage de requête de Neo4j. Il décrit des **patterns** de graphe avec une syntaxe visuelle :

```
(node)          -- un node
-[relation]->   -- une relation dirigée
(a)-[r]->(b)    -- a est relié à b via r
```

### Syntaxe fondamentale

```cypher
-- Lire des nodes
MATCH (n:Label) RETURN n

-- Filtrer
MATCH (n:Entity) WHERE n.name = "Alice" RETURN n

-- Créer un node
CREATE (n:Person {name: "Alice", age: 30})

-- Créer une relation
MATCH (a:Person {name: "Alice"}), (b:Company {name: "Orpheam"})
CREATE (a)-[:WORKS_AT {since: 2024}]->(b)

-- Supprimer (DETACH supprime aussi les relations)
MATCH (n:Person {name: "Alice"}) DETACH DELETE n
```

---

## Requêtes de base — Explorer le graphe

### Voir tout (limité)
```cypher
MATCH (n) RETURN n LIMIT 100
```

### Compter les nodes par label
```cypher
MATCH (n)
RETURN labels(n) AS type, count(*) AS total
ORDER BY total DESC
```

### Compter les relations par type
```cypher
MATCH ()-[r]->()
RETURN type(r) AS relation, count(*) AS total
ORDER BY total DESC
```

### Voir les index et contraintes
```cypher
SHOW INDEXES
SHOW CONSTRAINTS
```

---

## Schéma Graphiti dans Neo4j

Graphiti crée automatiquement cette structure :

| Label | Rôle | Propriétés clés |
|-------|------|-----------------|
| `Entity` | Entités extraites (personnes, entreprises, technos...) | `name`, `summary`, `group_id`, `embedding` |
| `Episodic` | Texte source ingéré | `name`, `content`, `source_description`, `reference_time` |
| `Community` | Clusters d'entités liées | `name`, `summary` |

### Relations

| Type | Entre | Propriétés clés |
|------|-------|-----------------|
| `RELATES_TO` | Entity → Entity | `fact`, `valid_at`, `expired_at`, `embedding` |
| `MENTIONS` | Episodic → Entity | Lie un épisode aux entités qu'il a générées |
| `HAS_MEMBER` | Community → Entity | Appartenance à une communauté |

### Temporalité

Graphiti ne supprime **jamais** de données. Quand un fait devient obsolète :
- `valid_at` = date où le fait est devenu vrai
- `expired_at` = date où il est devenu faux (`null` si toujours valide)

---

## Requêtes Graphiti — Entités

### Lister toutes les entités
```cypher
MATCH (n:Entity)
RETURN n.name, n.summary, n.group_id
ORDER BY n.name
```

### Chercher une entité par nom
```cypher
MATCH (n:Entity)
WHERE n.name CONTAINS "Alice"
RETURN n
```

### Chercher par nom (insensible à la casse)
```cypher
MATCH (n:Entity)
WHERE toLower(n.name) CONTAINS "alice"
RETURN n.name, n.summary
```

### Entités d'un groupe spécifique
```cypher
MATCH (n:Entity)
WHERE n.group_id = "test-graphiti-litellm"
RETURN n.name, n.summary
```

---

## Requêtes Graphiti — Relations (faits)

### Voir tous les faits entre entités
```cypher
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
RETURN a.name AS source, r.fact, b.name AS cible, r.valid_at, r.expired_at
LIMIT 50
```

### Faits encore valides (non expirés)
```cypher
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
WHERE r.expired_at IS NULL
RETURN a.name, r.fact, b.name, r.valid_at
```

### Faits expirés (historique)
```cypher
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
WHERE r.expired_at IS NOT NULL
RETURN a.name, r.fact, b.name, r.valid_at, r.expired_at
ORDER BY r.expired_at DESC
```

### Faits valides à une date donnée
```cypher
MATCH (a:Entity)-[r:RELATES_TO]->(b:Entity)
WHERE r.valid_at <= datetime("2024-03-01T00:00:00Z")
  AND (r.expired_at IS NULL OR r.expired_at > datetime("2024-03-01T00:00:00Z"))
RETURN a.name, r.fact, b.name
```

---

## Requêtes Graphiti — Episodes

### Lister les épisodes ingérés
```cypher
MATCH (e:Episodic)
RETURN e.name, e.source_description, e.reference_time
ORDER BY e.reference_time
```

### Voir quelles entités un épisode a généré
```cypher
MATCH (e:Episodic {name: "episode_1"})-[:MENTIONS]->(n:Entity)
RETURN e.name, n.name, n.summary
```

### Voir tous les épisodes qui mentionnent une entité
```cypher
MATCH (e:Episodic)-[:MENTIONS]->(n:Entity {name: "Alice Dupont"})
RETURN e.name, e.content, e.reference_time
```

---

## Requêtes Graphiti — Contexte d'une entité

### Voisinage direct (1 saut)
```cypher
MATCH (n:Entity {name: "Alice Dupont"})-[r]-(m)
RETURN n, r, m
```

### Voisinage étendu (2 sauts)
```cypher
MATCH path = (n:Entity {name: "Alice Dupont"})-[*1..2]-(m)
RETURN path
LIMIT 50
```

### Résumé du contexte d'une entité
```cypher
MATCH (n:Entity {name: "Alice Dupont"})-[r:RELATES_TO]-(m:Entity)
WHERE r.expired_at IS NULL
RETURN m.name, r.fact
```

---

## Requêtes avancées

### Trouver le chemin le plus court entre deux entités
```cypher
MATCH path = shortestPath(
  (a:Entity {name: "Alice Dupont"})-[*]-(b:Entity {name: "Orpheam"})
)
RETURN path
```

### Entités les plus connectées (hubs)
```cypher
MATCH (n:Entity)-[r]-()
RETURN n.name, count(r) AS connexions
ORDER BY connexions DESC
LIMIT 10
```

### Communautés et leurs membres
```cypher
MATCH (c:Community)-[:HAS_MEMBER]->(n:Entity)
RETURN c.name, c.summary, collect(n.name) AS membres
```

### Statistiques globales du graphe
```cypher
MATCH (n) WITH count(n) AS nodes
MATCH ()-[r]->() WITH nodes, count(r) AS rels
RETURN nodes, rels
```

### Supprimer un groupe de test
```cypher
-- /!\ Destructif — supprime toutes les données d'un group_id
MATCH (n) WHERE n.group_id = "test-graphiti-litellm"
DETACH DELETE n
```

---

## Tips Neo4j Browser

- **Visualisation** : cliquer sur un node pour voir ses propriétés
- **Expand** : double-cliquer sur un node pour voir ses voisins
- **Styling** : cliquer sur un label en haut pour changer couleur/taille
- **Export** : bouton screenshot en haut à droite pour exporter en PNG/SVG
- **`:help`** : taper `:help` dans la barre pour l'aide intégrée
- **`:schema`** : affiche le schéma du graphe (labels, types de relations)
- **Auto-complete** : `Tab` pour compléter les labels et propriétés
