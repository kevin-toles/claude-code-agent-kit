Assess taxonomy coverage and impact against enriched metadata and knowledge base content. Use when taxonomy changes must be validated for coverage, quality, and evidence.

$ARGUMENTS is the absolute path to the taxonomy file to assess.

# Taxonomy Impact Assessment

## When To Use
Use this skill immediately after taxonomy creation or updates (`/taxonomy`) to quantify coverage and identify weak taxonomy leaves.

## Workflow
1. Confirm `taxonomy_path` for the target taxonomy artifact.
2. Call `analyze_taxonomy_coverage(taxonomy_path=<path>)`.
3. Use optional parameters for collection, threshold, and top_k when user specifies strictness.
4. Summarize score, weak nodes, and evidence-backed remediation priorities.
5. Feed findings back into taxonomy update workflow.

## Tool Contract
- Tool: `analyze_taxonomy_coverage`
- Required: `taxonomy_path`
- Optional: `output_path`, `collection`, `top_k`, `threshold`, `max_leaf_nodes`, `subtree_root`, `concurrency`, `include_evidence`, `scoring_weights`

## Output Format
Return:
1. Coverage score
2. Lowest-performing taxonomy areas
3. Evidence summary
4. Proposed update actions

## Guardrails
- Require explicit `taxonomy_path`.
- Include evidence when available; do not report unsupported conclusions.
