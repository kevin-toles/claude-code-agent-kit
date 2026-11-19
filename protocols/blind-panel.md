# Blind LLM panels for building evaluation sets

LLMs are useful for labelling evaluation data at a scale one person cannot grade, and they are dangerous for it too: models from one family share biases, and they hallucinate confidently. This protocol uses them as graders while keeping the decision with evidence.

**Roles.**
- The **proposer** finds candidates and never grades them.
- **Two graders** (pinned model versions, one agent definition each; see `agents/`) grade every item blind: they get the item sheet only, never each other's verdicts or the proposer's reasoning.
- A **third grader** sees only the items the two disagree on. The final label is the majority, and the third grader breaks three-way ties.

**Ground truth comes from people, not the panel.** In the search golden set, a question's key points come only from human-vetted answers (accepted, or scored 10 or more), each citing its answer id. Candidates are found by plain keyword search, never by the embedding search being evaluated, so no model under test is favoured.

**Pre-register every gate.** Before a cheaper or local model is given a seat, the bar is written down: agreement with the existing graders of at least a fixed fraction. In one run, two local models reached 0.43–0.48 against a 0.61 bar and were dropped.

**Check the panel against human decisions.** Two checks, both against things people decided:
- A passage that a vetted answer cites as its source must be graded relevant.
- The panel's subject label must match the question's community tags, wherever a tag maps to exactly one subject.

**Evidence decides, not the majority.** For any golden set or label audit, the deciding factor is alignment with real evidence retrieved from the corpus, the raw data or reference code. Graders compare each item against that evidence and cite file and line, and the citations are checked mechanically. Generic lines that match everywhere (`}`, `else`) don't count as evidence. Results are reported in three groups: items with evidence, items without evidence (panel only), and items where the panel contradicts the evidence.

**Operational notes.**
- Pin the model in each agent file's frontmatter. New agent files load only in a fresh session.
- One grader sometimes skips items or invents taxonomy labels. Validate every verdict file against the item sheet and the label vocabulary, and issue make-up sheets for gaps.
- Cap concurrent subagents and expect usage limits on large runs; resume rather than restart.
