# Pipeline V3 — Approche Neuro-Symbolique

Le pipeline combine deux paradigmes :
- **Neuro** (LLM) : extraction flexible, comprend le langage naturel
- **Symbolique** (ASP + GNN) : raisonnement logique, contraintes formelles, cohérence garantie

## GNN — Graph Neural Network Légère

### C'est quoi ?

Un réseau de neurones sur graphe ultra-léger (1 couche, inference-only, pas d'entrainement). Il calcule un **embedding contextuel** pour chaque entité candidate en agrégeant les informations de ses voisins dans Neo4j.

### Comment ca marche

```
1. Récupère le voisinage de l'entité dans Neo4j (profondeur 2, max 200 noeuds)
2. Encode l'entité et ses voisins avec SentenceTransformer (all-MiniLM-L6-v2, 384 dims)
3. Agrège : h = mean([embedding_centre, mean(embeddings_voisins)])
4. Normalise L2
```

**Fallback** : si SentenceTransformer n'est pas installé → hash-based embedding déterministe (reproductible).

### A quoi ca sert

L'embedding capture le **contexte structurel** de l'entité. Une entité bien connectée dans le graphe aura un embedding différent d'une entité isolée. Ce signal est utilisé par l'Adaptive Fusion pour calculer un score de cohérence.

---

## Adaptive Fusion — Du vecteur au texte

### C'est quoi ?

Le pont entre le GNN (vecteurs numériques) et le LLM (texte). Convertit les embeddings GNN en **conditions sémantiques textuelles** injectées dans les prompts.

### Comment ca marche

1. **Score de cohérence** : projection aléatoire fixe (seed=42) de l'embedding → scalaire → sigmoide → score [0, 1]
   - Score élevé (>0.7) = entité bien connue et liée au graphe
   - Score faible = nouvelle occurrence ou peu référencée

2. **Analyse du voisinage** : types de voisins dominants (top 3)

3. **Formatage textuel** :
```
[ Alice Johnson ]
  → Type(s) voisin(s) : Person (5), Organization (3)
  → Voisinage KG (7 noeuds, profondeur 2) :
    - TechCorp --[WORKS_AT]--> Alice Johnson
  → Entités liées : TechCorp, Paris
  → Cohérence : 0.78 (élevée)
```

4. **Injection dans les prompts** : max 10 conditions, triées par cohérence décroissante

---

## ASP — Answer Set Programming

### C'est quoi ?

[ASP](https://potassco.org/clingo/) est un paradigme de programmation logique déclarative. On déclare des **faits** et des **regles**, et le solveur (clingo) trouve toutes les solutions (answer sets) qui respectent les contraintes.

### Comment ca marche dans le pipeline

**1. Traduction** — Les entités/relations extraites sont converties en faits ASP :
```prolog
entity("alice_johnson", "person").
confidence_entity("alice_johnson", 95).
relation("works_at", "alice_johnson", "techcorp").
existing_entity("techcorp", "organization").    % déja dans Neo4j
```

**2. Regles** — 3 fichiers de regles chargés :

### base_integrity.lp — Contraintes universelles
- Pas de relation réflexive (entité → elle-meme)
- Pas de relation orpheline (source/cible inconnue)
- Confiance dans les bornes [0, 100]
- Pas d'entité au nom vide

### schema_enforcement.lp — Typage et ontologie
- Types valides : person, organization, location, technology, project, concept, event, document, product, generic
- Conflits de type : person ≠ organization, person ≠ location, etc.
- Contraintes structurelles par relation :
  - `WORKS_AT` : Person → Organization
  - `LOCATED_IN` : * → Location
  - `MANAGES` : Person → Person|Project

### domain_rules.lp — Déductions métier
- Marquage faible confiance (entités < 60%, relations < 50%)
- Déduction de collègues (meme organisation)
- Localisation indirecte (personne travaille dans org localisée → probablement meme lieu)
- Détection entités isolées (0 relations)
- Transitivité MANAGES (confiance >= 70%)

**3. Réparation** — Chaque violation est corrigée automatiquement :

| Violation | Stratégie |
|-----------|-----------|
| Relation réflexive | Supprimée |
| Relation orpheline | Supprimée |
| Conflit de type | Garder l'entité a confiance max |
| Violation de schéma | Downgrade confiance -0.2 + flag warning |
| Confiance hors bornes | Normalisée a [0.0, 1.0] |

---

## Pourquoi cette approche ?

Le LLM seul fait des erreurs :
- Il invente des relations qui n'ont pas de sens (reflexives, orphelines)
- Il attribue des types incohérents (Alice = Person ET Organization)
- Il ne respecte pas les contraintes de schéma (WORKS_AT entre deux Location)

L'ASP corrige ca **formellement** — pas avec un autre appel LLM qui peut faire les memes erreurs, mais avec de la **logique pure** qui garantit la cohérence.

Le SCT/GNN améliore l'extraction en **informant** le LLM du contexte graphe existant, ce qui réduit les erreurs en amont.

```
Sans neuro-symbolique : LLM → erreurs → stockage d'incohérences
Avec neuro-symbolique : contexte graphe → LLM mieux informé → vérification logique → stockage propre
```
