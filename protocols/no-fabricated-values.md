# No fabricated values

**The failure it fixes.** A coding agent wired a graph-neighbour lookup through a vector-store scroll call. Scroll takes no query vector, so there was no similarity to return. The adapter hard-coded `score: 1.0` on every neighbour and returned them in storage order, and the rank-fusion step downstream read that order as relevance. Nothing errored. The follow-up audit found more of the same: a route reporting a capped term count as a knowledge-base match count, synthetic cascade results, and a seeding step writing all-zero vectors "to be filled later" that nothing ever filled.

**The rule.** Never write or pass through a value that looks like a measurement but is not one:

- a literal score or confidence on something nothing scored;
- a zero, random or placeholder vector;
- a constant "confidence";
- a count that was never queried;
- a neutral pass when there is no data.

If the API being called cannot produce the value, change the call so it can (for example, a filtered vector search instead of a scroll). If no call can, stop and raise it as an open question. Never fill the field.

**How to audit for it.** In non-test code, grep for literal scores and confidences, zero vectors, `unwrap_or(<float>)`-style defaults, and the words placeholder, simplified, heuristic and "for now". When reusing an existing adapter, read what it fills in before wiring it.

**The same rule for documents.** A decision document contains no number that was not measured. Worked examples come from running the real code on real data, and each statement is labelled measured, measured by reading the code, expected, or illustrative.
