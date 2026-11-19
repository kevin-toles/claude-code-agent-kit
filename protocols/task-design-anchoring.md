# Task → design-line anchoring

**The failure it fixes.** In an audit of one multi-repository feature, 228 work-breakdown rows that coding agents had marked Complete did not conform to the design. Another 34 tasks carried the wrong acceptance-criteria tag. The agents had read the task title, done something reasonable, and reported success. The code compiled and the tests passed.

**The rule.** An agent may not change a task's status until it has done all five steps, in order, for that one task:

1. **Quote the design.** Find the exact lines in the design document that the task implements (the behaviour, contract, pseudocode or file-manifest row) and quote them. A task's tag is never taken at face value; the requirement comes from the design.
2. **RED on real shapes.** Write a failing test against real data: recorded fixtures or real files, never invented shapes. Show the failure.
3. **GREEN on the quote only.** Implement exactly what the quoted lines say, nothing more. Show the pass.
4. **Compare line by line.** Check the code against the quote, line by line.
5. **Record an evidence row.** Test output plus `file:line`. Only then does the status change.

One task's five steps finish before the next task starts.

**Checks that catch "done" that isn't.**
- A helper extracted in a refactor with zero production callers means the task is not done.
- A module imported from more than one entry point is import-tested from each one.
- The whole test directory runs, not a `-k` filter.
- A validator PASS is not completion. Cross-check the manifest, the affected components, data artefacts, open questions and benchmarks.

**When a decision is made,** it closes in one pass: design document and work breakdown, test-first code in every affected repository, then validation and tests. A decision stays documented but unbuilt only when the data it needs does not exist yet, and its task then names that blocker.
