# Claude Code agent kit: quality gates for AI-written code

Most of my platform is written by AI coding agents, Claude Code above all. They are fast, and left alone they drift: in one audit, 228 work-breakdown tasks that agents had marked Complete did not match the design, and one adapter shipped a hard-coded "perfect" similarity score that downstream ranking read as real. The code compiled and the tests passed.

This kit is what I run on top of the agents so their work can be trusted: the rules they work under, the blind grader agents I use to build evaluation sets, and the skills that ground them in platform evidence before they touch code.

## Protocols

| Protocol | What it prevents |
|---|---|
| [Task → design-line anchoring](protocols/task-design-anchoring.md) | Agents marking work done that doesn't match the design: quote the design lines, RED on real data shapes, GREEN on the quote only, line-by-line check, evidence row, and only then a status change |
| [No fabricated values](protocols/no-fabricated-values.md) | Placeholder scores, zero vectors and invented counts posing as measurements |
| [Blind LLM panels](protocols/blind-panel.md) | Trusting one model family's shared bias when building evaluation sets: proposer never grades, two pinned blind graders, a third on disagreements, pre-registered seat gates, checks against human decisions, evidence over majority |
| [Decision documents](protocols/decision-docs.md) | Choosing between agent-generated options on guesses: every option gets a measured worked example a product owner can read |

## Agents

Each file is a Claude Code subagent definition with a pinned model. I keep one copy per model (for example `panel-grader-sonnet-5-5`, `panel-grader-opus-4-8`, `panel-grader-fable-5-2`) so that a panel is made of genuinely different graders. One example of each role is included.

| Agent | Job |
|---|---|
| [panel-grader](agents/panel-grader-sonnet-5-5.md) | Grades a retrieved passage 0–3 against a question's human-vetted key points, and files it in the subject taxonomy |
| [pair-grader](agents/pair-grader-sonnet-5-5.md) | Judges whether two programming-problem statements are the same problem, the same pattern, related or different |
| [label-auditor](agents/label-auditor-opus-5-5.md) | Audits a code-line answer key: is the target line code, not code, or ambiguous? |
| [block-profiler](agents/block-profiler-opus-5-5.md) | Tags extracted code blocks by language and kind |

All of them read a sheet and write a verdicts JSON with only the Read and Write tools: no shell, no network, no access to other graders' output. Results produced with them are in [etdi-decision-evals](https://github.com/kevin-toles/etdi-decision-evals) and [callgraph-accuracy](https://github.com/kevin-toles/callgraph-accuracy).

## Skills

Slash-command skills that sequence an agent through the platform's tools, which are served by [mcp-gateway](https://github.com/kevin-toles/mcp-gateway), before it changes code:

| Skill | Sequence |
|---|---|
| [dev-workflow](skills/dev-workflow.md) | Research in the knowledge base → map the architecture → audit only the risky files → remediation plan |
| [code-audit](skills/code-audit.md) | Quality, security and dependency scans with evidence per finding |
| [search](skills/search.md) | Retrieval across textbooks, code, patterns and diagrams |
| [wbs](skills/wbs.md) | Work breakdown from a design document via a five-round multi-agent protocol |
| [taxonomy-impact](skills/taxonomy-impact.md) | Taxonomy coverage and impact against the knowledge base |

The skills call tools on my own platform, so they are examples of the pattern rather than drop-in commands.

## License

MIT
