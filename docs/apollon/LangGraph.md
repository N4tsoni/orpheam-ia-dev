# LangGraph — Agent a états

## C'est quoi ?

[LangGraph](https://langchain-ai.github.io/langgraph/) est un framework pour construire des **agents a états** (state machines) avec des LLM. Chaque étape est un "noeud" qui modifie un état partagé.

## Principe

```python
# On définit un état
class AgentState(TypedDict):
    user_input: str
    intent: str
    response: str

# On crée des noeuds (fonctions)
def classify(state):
    return {"intent": "kg_query"}

def respond(state):
    return {"response": "..."}

# On les connecte dans un graphe
graph = StateGraph(AgentState)
graph.add_node("classify", classify)
graph.add_node("respond", respond)
graph.add_edge("classify", "respond")
```

## Noeuds d'Apollon (11)

| Noeud | Rôle |
|-------|------|
| `intent_classifier` | Classifie la question (kg_query / general / direct) |
| `kg_awareness` | Evalue si le KG est pertinent pour la query |
| `query_decomposition` | Découpe en sous-tâches |
| `intelligent_router` | Choisit la route (FULL_KG / HYBRID / DIRECT / NO_MATCH) |
| `ner_extraction` | Extrait les entités nommées de la question |
| `semantic_retrieval` | Recherche vectorielle dans Neo4j |
| `ranking` | Score et classe les résultats KG |
| `context_building` | Formate le contexte pour le LLM |
| `llm_call` | Appelle le LLM (OrpheamRouter) |
| `result_aggregator` | Combine les résultats de sous-tâches |
| `memory_persist` | Sauvegarde l'historique (Redis) |

## State

```python
class AgentState(TypedDict):
    conversation_id: str
    user_input: str
    intent: str                          # "kg_query" | "general" | "direct"
    kg_match_score: float                # 0-1
    subtasks: list[Subtask]
    routing_decision: RoutingDecision    # FULL_KG | HYBRID | DIRECT | NO_MATCH
    messages: list[BaseMessage]
    extracted_entities: list[ExtractedEntity]
    kg_candidates: list[dict]
    ranked_context: list[RankedCandidate]
    response: str
```
