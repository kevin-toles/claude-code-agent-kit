Generate a Work Breakdown Structure using the WBS_GENERATION Kitchen Brigade protocol (5-round: architect → engineer+estimator → reviewer → finalizer → compliance auditor). All round content is processed silently; only a round-status summary table and the output file path are shown.

$ARGUMENTS is one of:
- A file path to a design doc, control plane doc, or any markdown spec to generate a WBS from
- A free-form description of the project/feature to break down
- Both: a file path followed by additional constraints or scope notes

If $ARGUMENTS is omitted, ask for a source doc or description before proceeding.

# WBS Generation Workflow

## Step 1 — Resolve input

If $ARGUMENTS contains a file path (starts with `/` or `~`, or ends with `.md` / `.json`):
- Read the file
- Use its full content as `project_description`
- Note the filename in the submission for context

If $ARGUMENTS is a free-form description:
- Use it directly as `project_description`

If both a path and additional text are provided:
- Prepend the file content, append the extra text as "Additional constraints/scope:"

## Step 2 — Submit to WBS_GENERATION protocol

Call `a2a_send_message` with:

```
content = "WBS_GENERATION|" + JSON.stringify({
  "project_description": <resolved input from Step 1>,
  "constraints": [],
  "team_size": 1
})
skill_id = ""
context_id = ""
```

This returns a `task_id` immediately (202 Accepted). The 5-round protocol runs in the background.

## Step 3 — Poll for completion

Call `a2a_get_task(task_id=<task_id>)` with backoff:
- Poll every 30s — WBS_GENERATION typically takes 3–8 minutes
- Status values: `pending` | `running` | `completed` | `failed`
- On `completed`: the response contains the output artifact and file path
- On `failed`: report the error and offer to retry

Do NOT show round-by-round content to the user. The `output_mode` is `file_only`.

## Step 4 — Report output

Show only the round-status summary table and the output file path:

| Round | Role | Verdict |
|---|---|---|
| 1 | ARCHITECT | {phase_count} phases, valid DAG ✅ |
| 2 | ENGINEER + ESTIMATOR | {task_count} tasks, TDD paired ✅ |
| 3 | REVIEWER | {check_count} checks passed ✅ |
| 4 | FINALIZER | Written to file ✅ |
| 5 | COMPLIANCE AUDITOR | {verdict} ({score}/100) ✅ |

Then: `WBS written to: <output_file_path>`

If Round 5 score < 100, the protocol loops back to Round 4 automatically (up to 7 retries). Report the final verdict only.

## Step 5 — Remediate failures (if Round 5 < 100 after retries)

When the protocol exhausts retries and the WBS still has failures, enter the iterative remediation loop.

### 5.1 — Validate via direct HTTP (MANDATORY)

ALWAYS validate by running this bash command — it completes in under 100ms:

```bash
curl -s -X POST http://localhost:8091/validate/wbs \
  -H "Content-Type: application/json" \
  -d "$(jq -Rs '{content: .}' < '<WBS_FILE_PATH>')" \
  | python3 -m json.tool > /tmp/wbs_validation.json
```

Then read ONLY the result JSON to understand failures:

```bash
cat /tmp/wbs_validation.json
```

NEVER read the WBS file into your context to pass it to the `validate_wbs` MCP tool. NEVER spawn a sub-agent for validation. The MCP tool path reads 145KB+ into context and takes 10+ minutes. The curl path takes 99ms.

### 5.2 — Read and trust the `detail` field

Every `CheckFailure` in the result contains a `detail` field with the exact remediation instruction — correct column names, expected format, what to fix. ALWAYS read and follow the `detail` field. NEVER guess at correct formats, column names, or content structure.

### 5.3 — Fix one failure class at a time

Triage order (fix in this sequence):
1. **Structural** (T1, T2, T4) — missing TDD pairing, RED→RED sequences, task ordering
2. **Naming** (A-5) — "and" in task descriptions (split into separate tasks)
3. **Parameterization** (B-1, B-2, B-3) — hardcoded ports, paths, literals
4. **Perry** (PERRY.RISK, PERRY.REQ, PERRY.TEST, PERRY.COMP, PERRY.NET, PERRY.MOCK) — missing compliance sections
5. **Semantic** — remaining check classes
6. **WARNINGs** — only after all ERRORs are resolved

Fix ALL instances of one `check_id` in a single edit pass, then re-validate before moving to the next `check_id`.

### 5.4 — Known cascading fix patterns

Fixes are not isolated — one fix class frequently introduces another:

| Fix | Commonly Introduces | Prevention |
|---|---|---|
| **A-5 splits** (splitting "and" tasks into two) | T2 violations (RED→RED without GREEN between them) | When splitting a RED task, insert a GREEN task between the two new RED tasks |
| **GREEN task insertions** (fixing T2) | Duplicate task IDs (ID3) | Renumber ALL subsequent task IDs in the section after inserting |
| **Section renaming** | Broken Perry table references | Check that Perry tables still reference the correct section header |
| **Task ID renumbering** | Broken AC references | Update AC ID cross-references in the same edit pass |

### 5.5 — Re-validate after each fix class

Run the curl command from 5.1 again after each batch of fixes. Compare the new score to the previous score. If the score decreased, you introduced new failures — check what changed.

### 5.6 — Recovery from failed attempts

If a validation attempt fails (rate limit, timeout, interruption), retry the curl command immediately. Do NOT re-read the WBS file, copy it to a scratchpad, check file sizes, or do any other prep work — the file has not changed. Just run the curl command again.

### 5.7 — Exit criteria

Loop Steps 5.1–5.5 until: `verdict: "PASS"`, `score: 100`, `failures: []`.
Report the final result to the user.

## Guardrails

- Never write WBS content manually outside of remediation — generation always goes through this protocol
- Never show round content in chat — `output_mode: file_only` is enforced by the protocol config
- The compliance auditor requires 100/100 — do not accept or show partial results as final
- Validation-service at `:8091` is the sole WBS validator — `validate_wbs_compliance.py` is retired and deleted
- ALWAYS validate via direct `curl` to `:8091`, NEVER by reading the file into context and using the MCP tool
- NEVER spawn a sub-agent for validation — run the curl command directly in bash
- NEVER attempt to fix all validation failures in one pass — fix one `check_id` class at a time, re-validate, then proceed
- ALWAYS read the `detail` field of each `CheckFailure` for exact remediation instructions — never guess
