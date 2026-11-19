# Open-question documents a product owner can decide from

Agents produce options quickly. Deciding between them is the slow part, and it is where a wrong guess costs a multi-day reload. These documents are written so that someone who does not read code can decide, and an engineer can check the mechanics.

1. **Written for the product owner.** For each question: the decision in plain words, why it matters to the people and agents using the platform, and for every option how the system behaves and what the user sees. Technical references go at the foot of each entry.
2. **Plain-language terms with a glossary.** For example, classification is filing a chapter on a subject shelf, seeding is loading, and ring search is widening to neighbouring shelves.
3. **A worked example for every option,** showing what an engineer or agent actually gets back: the chapters, code blocks, diagrams or responses.
4. **Examples are measured, not imagined.** They come from running the platform's real code on real data. In one round, measuring overturned several claims the document had made from illustrative examples.
5. **Every statement labelled:** measured, measured by reading the code, expected, or illustrative. Each example says what the simulation cannot reproduce. No unmeasured numbers.
6. **Grouped by what each question blocks:** decisions that block tasks; decisions the planned runs will settle; decisions that block nothing; on hold; found but not yet raised.
7. **Transient.** Each decision is written into the design document and the work breakdown; the file is deleted once nothing in it is undecided.

**One rule about answers.** Naming an option ("back to (a) for X") reopens a question; only an explicit choice ("go with (a)") records a decision. Recording a decision nobody made changes the design of record and has to be unwound.
