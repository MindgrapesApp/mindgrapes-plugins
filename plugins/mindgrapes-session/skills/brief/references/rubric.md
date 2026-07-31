# The capture rubric

One test governs everything:

> Would future-you, on a different project with a different AI, want this surfacing in
> semantic search?

Three words in that sentence carry the weight. **Different project**: it has to travel.
**Different AI**: no session context comes with it. **Semantic search**: someone has to
plausibly go looking for it.

If a candidate needs this repo, this branch, or this conversation to make sense, it fails.

## Capture

- **Transferable technique or gotcha.** A trap that would catch you again somewhere else, and
  the tell that gives it away.
- **A decision with reasoning that outlives the repo.** The reasoning is the durable part.
  A decision recorded without its "why" is trivia.
- **Facts about people, companies, relationships.** Who owns what, who to ask, what someone
  cares about, how a relationship stands.
- **Self-knowledge.** Discovered preferences, working style, what consistently goes wrong.
- **Outcomes another project would need.** Shipped, abandoned, blocked on someone, decided
  against and why.
- **Measurements that ruled something in or out.** The number matters more than the code that
  produced it.

## Skip

claude-mem already captures the per-repo layer automatically. These belong to it:

- File-level mechanics: function names, call sites, which module does what
- A local bug and its local fix
- Anything reconstructible by reading the repo at HEAD
- Test failures, red-to-green churn, intermediate debugging state
- Plans and next steps (those are issues or tasks, not memory)
- Restatements of something already in Mind Grapes

## Calibration

The same afternoon of work, split correctly.

**claude-mem's, not Mind Grapes':**

> Fixed the relationship lookup so it matches merge-normalized anchors; capped the hop limit
> at 3.

Function name, that schema, that pull request. Reading the repo tells you all of it.

**Mind Grapes':**

> Recursive graph traversal cost went exponential past hop 3 on a production-scale entity
> graph: 27ms at 2 hops, 152 seconds at 6. Cap hop count explicitly rather than trusting the
> query planner to stay linear as depth grows.

No repo nouns. It would help on a different graph, in a different language, years later.

**Also Mind Grapes':**

> Benchmarking read-path changes against a unit-test-sized database hides cost cliffs
> entirely. Keep a production-scale copy as a standing validation target for anything that
> touches a read path.

A practice, learned once, applicable everywhere.

## Borderline calls

- **A technique that felt obvious in hindsight.** Capture it. Hindsight is exactly what does
  not survive to the next project.
- **Something already half-known.** Search first. If the existing capture is vaguer, propose
  the sharper version and flag the overlap for the user to arbitrate.
- **A strong opinion formed mid-session.** Capture the opinion with what produced it. An
  opinion without evidence retrieves as noise.
- **A decision that might get reversed next week.** Skip it. Wait for it to settle. Mind
  Grapes is not a status board.

## The counting rule

Five captures per session is the ceiling, not the target. Sessions that genuinely produce
five are rare. Two is a good day. Zero is the most common honest answer, and reporting zero
costs nothing while a junk capture pollutes every future search that brushes past it.
