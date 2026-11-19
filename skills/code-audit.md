Run comprehensive code quality, security, dependency, and architecture scans using audit-service and struct-analyzer tools. Use when users ask for code audit, anti-pattern detection, security review, dependency health, or structural analysis.

$ARGUMENTS should be the absolute path to a service directory to scan. If omitted, ask for it.

# Code Audit and Structural Scans

## Service Roles

**struct-analyzer** (`:8088`, Go) operates on **directory paths** (`source_path`). It produces a structural map of the whole system: call graph, boundary map, component registry, event/messaging topology, dead code inventory. Run struct-analyzer first — it tells you *where* in the system the risks live.

**audit-service** (`:8084`) operates on **file content** (`code` parameter). It scores individual files against rules across 5 pillars: CP (structural), SA (architectural smells), AP (anti-patterns), SEC (security), EL (eloquence). Run audit-service second — against the specific files struct-analyzer identified as high-risk.

**Exception:** `audit_dependency_assess(source_path=...)` and `audit_codebase_scan(source_path=...)` take a directory, not file content.

The sequencing is always: **struct-analyzer → identify hot spots → audit-service → scan those files**.

## Workflow 1: Full Service Health Check

```
Step 1 — Architecture Snapshot (struct-analyzer, source_path)
  sa_architecture_mapping_log(source_paths=[<service_dir>])
  → Component Registry: classes, modules, routes
  → Communication Map: outbound HTTP, event emitters, messaging consumers

Step 2 — Structural Analysis (struct-analyzer, source_path)
  sa_detect_patterns(source_path=<service_dir>)    ← design patterns in use
  sa_detect_boundaries(source_path=<service_dir>)  ← layer violations
  sa_build_call_graph(source_paths=[<service_dir>]) ← fan-in/fan-out hot spots
  NOTE: sa_build_call_graph takes source_paths[] (list), not source_path

Step 3 — Quality Scan (Audit, code=<file_contents>)
  For each critical file identified by struct-analyzer:
  audit_quality_scan(code=<file_contents>, language="python")
  → CP001–CP018: function length, params, complexity, nesting
  → SA001–SA015: testability smells, layer boundary violations, over-mocking
  → AP001–AP005: Blob class, lava flow, boat anchor, redundant wrapper

Step 4 — Code Metrics (Audit, code=<file_contents>)
  audit_code_metrics(code=<file_contents>, pillars=["structural", "architectural", "eloquence"])
  → CC, Cognitive Complexity, CBO, RFC, LCOM, Maintainability Index

Step 5 — Dependency Health (Audit, source_path)
  audit_dependency_assess(source_path=<service_dir>)
  → DEP_DEAD: packages declared but never imported
  → DEP_HEAVY: large dependency used minimally
  → DEP_PASSTHROUGH: import-only-to-re-export smell
  → DEP_LOW_RATIO: package contributes 1–2 symbols despite being a large dep
  → DEP_UNSTABLE: high instability score per Martin metrics
  → DEP_FACADE_CANDIDATE: package structure suggests it should be wrapped
  → Returns dependency_health_score [0,100], zone: healthy / tolerable / questionable / bloated

Step 6 — Resolve High-Severity Findings (Audit)
  audit_resolve_lookup(violation_type=<violation_from_step3>)
  → Remediation principle + effort estimate + code fix pattern
```

## Workflow 2: Security + Architecture Risk Assessment

```
Step 1 — Find External Interfaces (struct-analyzer)
  sa_detect_events(source_path=<service_dir>)
  → HTTP routes, REST entry points, inbound surface

  sa_detect_messaging(source_path=<service_dir>)
  → Event emitters, message bus consumers, async producers

Step 2 — Trace Data Flow Paths (struct-analyzer)
  sa_build_call_graph(source_paths=[<service_dir>])
  → Which functions handle untrusted input?
  → Paths that reach DB queries, file writes, subprocess calls

Step 3 — Deep Security Scan on Hot-Path Files (Audit, code=<file_contents>)
  For each file identified in Step 2:
  audit_security_scan(
    code=<file_contents>,
    language="python",
    mode="deep",
    severity_threshold="low"
  )
  → SEC001–SEC022 violations with CWE IDs and confidence scores

Step 4 — Exploit Context (Audit VRE)
  audit_search_exploits(
    query=<cwe_description_from_step3>,
    cwe_ids=["CWE-78", "CWE-89"],
    min_similarity=0.7
  )
  → Ranked ExploitMatch objects from VRE quarantine (Qdrant :6336)

Step 5 — CVE Lookup (Audit VRE)
  audit_search_cves(cwe_id="CWE-89", ecosystem="python", severity="high")
  → CVERecord objects with CVSS scores and reference URLs
```

## Workflow 3: Dead Code and Dependency Pruning

```
Step 1 — Dead Code Discovery (struct-analyzer)
  sa_detect_dead_code(source_path=<service_dir>)
  → Unused functions not called from anywhere in the source tree
  → Unused imports

Step 2 — Dependency Health (Audit)
  audit_dependency_assess(source_path=<service_dir>)
  → DEP_DEAD, DEP_HEAVY, DEP_PASSTHROUGH, DEP_LOW_RATIO, DEP_UNSTABLE

Step 3 — Cross-Validate via Call Graph (struct-analyzer)
  sa_build_call_graph(source_paths=[<service_dir>])
  → Confirm dead-code candidates have 0 callers
  → Identify orphan clusters (only call each other, never invoked from entry point)

Step 4 — Resolve (Audit)
  audit_resolve_lookup(violation_type="DEP_DEAD")
  → Distinguish dead-dead from intentional "future use" patterns
```

## Workflow 4: Full Codebase VRE Scan (Large Repos)

```
audit_codebase_scan(source_path=<service_dir>, priority_filter=["CRITICAL", "HIGH"])
  → Walks source tree recursively
  → Runs full 4-layer detection pipeline on each .py/.js/.ts/.go/.java file
  → Deduplicates findings by (pattern_id, file)
  → Enriches SEC* findings with VRE exploit evidence from Qdrant :6336
  → Use priority_filter to focus on critical/high findings first
```

## Audit-Service Rule Reference

| Series | Pillar | What it catches |
|--------|--------|----------------|
| CP001–CP018 | structural | Function length, params, complexity, nesting, naming |
| SA001–SA015 | architectural | Testability smells, layer violations, IoC misuse, over-mocked tests |
| AP001–AP005 | architectural | Blob class, lava flow, boat anchor, redundant wrapper |
| SEC001–SEC022 | security | Hardcoded secrets, SQL injection, weak crypto, CORS, subprocess injection |
| EL001–EL003 | eloquence | Verbosity, comment ratio, cognitive load |

## Output Format
Return:
1. struct-analyzer structural findings — patterns, boundaries, event/messaging topology, hot spots, dead code
2. Audit findings per file — ordered by severity with CWE IDs and pillar scores
3. Exploit and CVE context for security findings
4. Dependency health score and zone (healthy / tolerable / questionable / bloated)
5. Remediation plan: violation type → fix pattern → effort estimate

## Guardrails
- Always run struct-analyzer first to identify which files to audit — do not scan all files blindly
- `audit_quality_scan`, `audit_security_scan`, `audit_code_metrics` take `code` (file content), not a path
- `audit_dependency_assess` and `audit_codebase_scan` take `source_path` (directory)
- `sa_build_call_graph` takes `source_paths` (list) — different from all other sa_* tools
- Include test files in struct-analyzer `source_path` to avoid false dead-code positives
- struct-analyzer cannot detect dynamic dispatch (`getattr`, `importlib`) — note this in output
- `amve_*` tools are deprecated — always use `sa_*` equivalents
