Orchestrate end-to-end development workflows by combining unified search research, struct-analyzer architecture mapping, and audit-service scanning. Use when users ask for implementation planning, architecture-aware reviews, risk triage, or remediation plans grounded in platform evidence.

$ARGUMENTS is the task or question to orchestrate around. If omitted, ask for it.

# Development Orchestrator Workflow

## When To Use
Use this skill when a development task needs evidence-driven research, architecture analysis, and code quality or security auditing combined in a single workflow. Typical triggers: implementation planning, pre-refactor risk triage, security review, new codebase onboarding, or producing a remediation backlog.

## Core Services and Their Roles

| Service | Port | Input | What It Answers |
|---------|------|-------|-----------------|
| Unified Search (facade) | :8081 | `query` | What does correct implementation look like? What do the textbooks say? |
| struct-analyzer | :8088 | `source_path` (directory) | What does this system look like structurally? Where are the coupling hotspots? |
| Audit Service | :8084 | `code` (file content) | What code-level violations exist in these specific files? |

**Sequencing is not optional.** Research first gives you domain knowledge. struct-analyzer second gives you structural topology and identifies which files carry risk. Audit third scans those specific files — not the whole codebase blindly.

## Phase 1 — Research and Context

Use the unified search facade to ground the task in platform knowledge before touching any code.

```
ask(query=<broad_question>, difficulty="intermediate")
  → Searches all shelves (textbooks + code + patterns + diagrams)
  → Use when you don't know where the answer lives

search_in(query=<specific_question>, source="textbooks")
  → Target a specific shelf when you know the domain
  → source options: "textbooks" | "code" | "patterns" | "diagrams"

find_code_pattern(query=<pattern_name>, examples="both")
  → Pull good and bad CRE implementation examples
  → Use to compare platform-canonical patterns against what struct-analyzer finds

graph_query(cypher=<cypher_query>)
  → Read-only Neo4j queries when topology relationships matter
  → e.g., "which services depend on this component?"

foundation_search(query=<first_principles_question>)
  → Scientific / theoretical grounding from unified-search (:8081)
  → Use for algorithm correctness, complexity, formal properties
```

> **Pending Rust USS upgrade:** At Rust USS cutover, `foundation_search` → `uss_scientific_search`, `graph_query` → `uss_graph_query`, `graph_traverse` → `uss_graph_traverse`. Until then, use the current Python-backed tools. `uss_ring_search` will offer higher-precision 6-ring concentric search as an alternative to `ask`.

**Do not** call `semantic_search` or `hybrid_search` directly — they require internal Qdrant collection names. Use the facade tools above.

## Phase 2 — Architecture Mapping (struct-analyzer)

Run struct-analyzer against the service directory. Always directory-in, structural-map-out — never file content.

```
Step 1 — Full Discovery
  sa_architecture_mapping_log(source_paths=[<service_dir>])
  → Full component registry + communication map
  → Use for new or unfamiliar codebases

Step 2 — Structural Analysis
  sa_detect_patterns(source_path=<service_dir>)
  → Architectural patterns in use; confidence scores show strength
  → Absent patterns are gaps (e.g., no hexagonal → expect coupling problems)

  sa_detect_boundaries(source_path=<service_dir>)
  → Layer violations: domain importing infra, cross-cutting concerns scattered

  sa_detect_events(source_path=<service_dir>)
  → HTTP routes, REST entry points — critical for security workflows
  → Replaces amve_detect_communication (event/HTTP side)

  sa_detect_messaging(source_path=<service_dir>)
  → Event emitters, message bus producers/consumers
  → Replaces amve_detect_communication (async/messaging side)

  sa_build_call_graph(source_paths=[<service_dir>])
  → Inter-function call graph; high fan-in/fan-out = coupling hotspot
  → Traces execution paths from entry points to sinks (DB, subprocess, file writes)
  → NOTE: takes source_paths[] (list), not source_path

Step 3 — Hygiene
  sa_detect_dead_code(source_path=<service_dir>)
  → Orphan functions and clusters never invoked from any entry point
```

**Output of this phase:** A ranked list of files and components that carry the highest architectural risk. These become the inputs for Phase 3.

## Phase 3 — Code and Security Audit (Audit Service)

Audit tools take **file content** (`code` parameter), not directory paths. Use the file list from Phase 2.

```
For each high-risk file identified by struct-analyzer:

  audit_quality_scan(code=<file_contents>, language=<lang>)
  → CP001–CP018: structural violations (length, complexity, nesting, naming)
  → SA001–SA015: architectural smells (testability, layer violations, IoC misuse)
  → AP001–AP005: anti-patterns (Blob class, lava flow, boat anchor)

  audit_security_scan(code=<file_contents>, language=<lang>, mode="deep", severity_threshold="low")
  → SEC001–SEC022: security violations with CWE IDs and confidence scores
  → Use mode="deep" for files that handle untrusted input (identified by sa_detect_events)

  audit_code_metrics(code=<file_contents>, pillars=["structural", "architectural", "eloquence"])
  → CC, LCOM, CBO, RFC, Maintainability Index
  → High CBO + SA012 = hardest files to test

For the whole directory (source_path, not file content):

  audit_dependency_assess(source_path=<service_dir>)
  → DEP_DEAD, DEP_HEAVY, DEP_PASSTHROUGH, Martin metrics

  audit_codebase_scan(source_path=<service_dir>, priority_filter=["CRITICAL", "HIGH"])
  → Full recursive scan with VRE exploit enrichment for SEC* findings
  → Use when you want a comprehensive baseline rather than targeted file scans

For security findings — add exploit context:

  audit_search_exploits(query=<cwe_description>, cwe_ids=[...], min_similarity=0.7)
  → Real exploit code from VRE quarantine (Qdrant :6336) — prioritizes findings with known weapons

  audit_search_cves(cwe_id=<cwe>, ecosystem=<lang>, severity="high")
  → Known CVE records with CVSS scores and reference URLs
```

## Phase 4 — Resolution and Remediation Plan

```
For top violations by severity:
  audit_resolve_lookup(violation_type=<violation_type_from_phase3>)
  → Remediation principle + effort estimate + code fix pattern
  → Distinguish quick wins from structural rework

For unresolved architecture questions:
  search_in(query=<refactoring_pattern>, source="patterns")
  find_code_pattern(query=<pattern>, examples="good")
  → Pull canonical implementation examples for the fix
```

## Phase 5 — Protocol Escalation via A2A (When Multi-LLM Processing Is Needed)

Some tasks require multi-LLM deliberation — code generation with review, adversarial quality checks, iterative debugging. These run as Kitchen Brigade protocols through the A2A bridge.

### How to Invoke a Protocol

```
Step 1 — Submit the task
  a2a_send_message(
    content="<protocol_id>|<json inputs>",
    skill_id="",
    context_id=""        ← optional: groups related tasks for threading
  )
  → Returns task_id immediately (202 Accepted — task runs in background)

Step 2 — Poll for completion
  a2a_get_task(task_id=<task_id>)
  → Returns status: pending | running | completed | failed
  → On completion: returns full protocol output artifact

Step 3 — Cancel if needed
  a2a_cancel_task(task_id=<task_id>)
```

### Kitchen Brigade Protocol Reference

| Protocol ID | When to Use | Flow |
|-------------|-------------|------|
| `PIPELINE_PROTOCOL` | Code generation, implementation from design | architect → implementer → reviewer → finalizer |
| `TROUBLESHOOTING_LOOP` | Iterative debugging — fix/test/retry cycles | decompose → cross_reference → 5-LLM discuss → classify blockers → propose fix → apply → test; loops on failure |
| `ARCHITECTURE_RECONCILIATION` | Conflicting architecture docs — find the canonical truth | 4-LLM parallel analysis → conflict synthesis → parallel resolutions → consensus |
| `DEBATE_PROTOCOL` | High-stakes design decisions — adversarial quality check | proposer vs opposer opening → rebuttal → moderator synthesis → judge verdict |
| `ROUNDTABLE_DISCUSSION` | Exploratory — get diverse perspectives on a problem | all participants parallel each round → converge on agreements/divergences |
| `FIVE_LLM_DISCUSSION` | Maximum reasoning diversity — complex problems | parallel analysis → cross-exam → deep dive → convergence → synthesizer + validator |
| `WBS_GENERATION` | Break down implementation tasks — produce a structured WBS | 5-round: architect → engineer+estimator → reviewer pitfall audit → finalizer → compliance auditor (100/100 required) |
| `RENAME_NORMALIZATION` | Normalize document filenames from content | scan/classify → extract content cues → analyze filename quality → rename suggestions |

### Guardrails for Protocol Use
- Use `TROUBLESHOOTING_LOOP` for bugs — it has built-in scope control and won't expand past Tier-1 issues without approval
- Use `PIPELINE_PROTOCOL` for code generation — it always includes a reviewer pass before the finalizer
- Protocol tasks are durable (Temporal-backed when available) — safe to start and poll later
- Use `context_id` to group multiple protocol runs in the same conversation thread

## Output Contract
Return:
1. Research evidence summary — key findings from Phase 1 that frame the task
2. Architecture findings — patterns detected, boundary violations, coupling hotspots, dead code
3. Audit findings per file — ordered by severity with CWE IDs and pillar scores
4. Exploit and CVE context for any security findings
5. Remediation backlog — violation → fix pattern → effort → verification step
6. Verification checklist — what to re-run after fixes are applied

## Guardrails
- struct-analyzer before audit: never run audit-service scans blindly across all files; let struct-analyzer identify which files to target
- `audit_quality_scan`, `audit_security_scan`, `audit_code_metrics` take `code` (file content), not paths
- `audit_dependency_assess`, `audit_codebase_scan`, and all `sa_*` tools take `source_path` (directory)
- `sa_build_call_graph` takes `source_paths` (list) — different from all other `sa_*` tools
- Never call `semantic_search` or `hybrid_search` directly — use `ask`, `search_in`, `find_code_pattern`
- Never use `amve_*` tools — they are deprecated; use `sa_*` equivalents
- Keep all findings grounded in tool evidence; separate confirmed facts from hypotheses
- For security workflows: sa_detect_events → identifies entry points → audit_security_scan on those files only
- `uss_*` tools return 503 until Rust USS cutover — use current Python-backed search tools
