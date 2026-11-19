Retrieve knowledge, examples, and architecture context using unified-search facade and expert tools. Use when users ask how something works, need targeted references, want code examples, or need graph and foundation-level retrieval.

$ARGUMENTS is the question or topic to search for.

# Information Retrieval — Unified Search

## When To Use
Use this skill for knowledge retrieval tasks across textbooks, code, patterns, diagrams, and graph relationships.

## Workflow
1. Start broad with `ask` for discovery questions.
2. Narrow with `search_in` when source shelf is known.
3. Use `find_code_pattern` for good, bad, or both implementation examples.
4. Use `diagram_search` for architecture visuals.
5. Use `graph_query` or `graph_traverse` for explicit topology relationships.
6. Use `foundation_search` for scientific or first-principles grounding.
7. Synthesize concise answer with cited evidence snippets.

## Preferred Tool Order

```
ask(query=<broad_question>, difficulty="intermediate")
  → Searches all shelves (textbooks + code + patterns + diagrams)
  → Use when you don't know where the answer lives

search_in(query=<specific_question>, source="textbooks")
  → Target a specific shelf when you know the domain
  → source options: "textbooks" | "code" | "patterns" | "diagrams"

find_code_pattern(query=<pattern_name>, examples="both")
  → Pull good and bad CRE implementation examples

diagram_search(query=<architecture_concept>)
  → Architecture diagrams and visual references

graph_query(cypher=<cypher_query>)
  → Read-only Neo4j queries for topology relationships
  → Routes to Python unified-search (:8081) — live now

graph_traverse(start_node=<node>, strategy="bfs")
  → BFS or MMR diversity-aware traversal from one or more starting nodes
  → Routes to Python unified-search (:8081) — live now

foundation_search(query=<first_principles_question>)
  → Scientific / theoretical grounding
  → Routes to Python unified-search (:8081) — live now
  → Will migrate to Rust USS (:8089) /v1/search/scientific at cutover
```

**Do not** call `semantic_search` or `hybrid_search` directly — they require internal Qdrant collection names. Use the facade tools above.

## Pending: Rust unified-search-rs (:8089)

These tools are registered but route to the pending Rust USS service. They return 503 until the Rust USS cutover:

```
uss_ring_search(q=<query>, collection="", mmr_strategy="")
  → 6-ring concentric ANN search: Ring 0 (ANN) → Rings 1–5 parallel → RRF fusion → MMR re-rank
  → Higher precision than single-collection search; use for complex retrieval

uss_scientific_search(query=<query>, top_k=10)
  → First-principles / theoretical retrieval (replaces foundation_search at cutover)

uss_embed(text=<text>)
  → Generate raw vector embedding for text

uss_hydrate(ids=[<vector_id>], collection="")
  → Fetch full document payloads by vector IDs

uss_fitness_evaluate(function_id="", function=<dict>, snapshot=<dict>)
  → Evaluate a single fitness function (mirrors sa_evaluate_fitness at the search layer)

uss_fitness_batch(requests=[...])
  → Evaluate multiple fitness functions in one call

uss_graph_query(cypher=<cypher_query>, params={})
  → Neo4j Cypher at the Rust USS layer (mirrors graph_query at cutover)

uss_graph_traverse(start_node=<node>, strategy="bfs", max_depth=3, top_k=10)
  → Graph traversal at the Rust USS layer (mirrors graph_traverse at cutover)
```

> **Note:** At Rust USS cutover, `foundation_search`, `graph_query`, and `graph_traverse` will redirect to their Rust equivalents. Until then, use the current Python-backed tools which are fully live.

## Output Format
Return:
1. Answer summary
2. Sources and snippets used
3. Contradictions or uncertainty notes
4. Recommended next query refinements

## Guardrails
- Prefer facade tools over raw internal collection calls
- Avoid broad unbounded searches when targeted refinement is possible
- `uss_*` tools return 503 until Rust USS cutover — fall back to current Python-backed tools
