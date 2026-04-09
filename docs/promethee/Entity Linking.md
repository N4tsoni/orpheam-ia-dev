# Entity Linking — Wikidata & DBpedia

## C'est quoi ?

L'entity linking connecte les entités extraites du KG a des **bases de connaissances externes** (Wikidata, DBpedia). Ca permet d'enrichir le graphe avec des identifiants universels et des informations complémentaires.

## Sources

### Wikidata
- API `wbsearchentities` — recherche par nom
- Retourne un identifiant Q (ex: `Q42` = Douglas Adams)
- Propriété ajoutée au noeud Neo4j : `wikidata_id`

### DBpedia Spotlight
- API `annotate` — annotation de texte
- Retourne des URIs DBpedia (ex: `dbpedia.org/resource/Douglas_Adams`)
- Propriété ajoutée : `dbpedia_uri`

## Fonctionnement

1. Récupère les entités depuis Neo4j (filtrables par type)
2. Pour chaque entité, cherche dans Wikidata et/ou DBpedia
3. Compare le nom avec un seuil de similarité (défaut : 0.7)
4. Ecrit les liens trouvés dans Neo4j

## Configuration

```python
EntityLinker(
    use_wikidata=True,
    use_dbpedia=True,
    min_similarity=0.7,
    entity_types=["Person", "Organization"]  # optionnel
)
```

## Résultat

```python
EntityLinkingResult(
    entities_processed=150,
    entities_linked=98,
    link_rate=0.65,
    sources_used=["wikidata", "dbpedia"]
)
```
