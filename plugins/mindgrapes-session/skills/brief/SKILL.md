---
name: brief
description: "Review the current Claude Code session and capture only what is worth remembering across projects into Mind Grapes. Invoke with /brief or when the user says 'brief the brain', 'anything worth capturing', 'wrap up the session', 'capture what we learned', or 'session brief'. Proposes candidates for approval rather than capturing silently, and proposing nothing is a normal outcome."
effort: low
---

# Session Brief

Sweep a working session, decide what future-you on a *different project with a different AI*
would want back, and capture only that into Mind Grapes.

The failure mode this skill exists to prevent is capturing everything. A brain full of
session logs is a worse brain. Most sessions produce zero or one capture. That is correct
behaviour, not a skill that failed to fire.

## When to Use This Skill

- User invokes `/brief`
- User says "brief the brain", "anything worth capturing?", "wrap up", "session brief"
- The user is closing out a session and wants the durable residue kept

Do **not** use this skill when the user says "remember THIS" about one specific thing. That
is a direct `capture_thought` call with their words, not a session sweep.

## Arguments

- `/brief` — sweep the current session
- `/brief <hint>` — sweep, but bias toward the named thread ("the traversal work")
- `/brief --auto` — skip the approval gate. Rubric and dedupe still apply.

## Division of Labour

Two memory stores, and this skill only feeds one:

- **claude-mem** already captures what happened on this repo. Automatic. Do not duplicate it.
- **Mind Grapes** holds durable, cross-project, cross-AI knowledge. Manual. That is this skill.

If a candidate is only meaningful inside this repo, claude-mem already has it. Drop it.

## Workflow

### Step 1: Establish provenance

```bash
git remote get-url origin 2>/dev/null | sed 's/.*[:/]\([^/]*\)\.git$/\1/' | sed 's/\.git$//'
git rev-parse --abbrev-ref HEAD 2>/dev/null
```

Fall back to `basename "$(pwd)"` when there is no remote. Hold these as `PROJECT` and
`BRANCH` for `source_ref`. Never put them in the capture text as a substitute for making the
text stand alone.

### Step 2: Sweep the session

Read back over the session for material that outlives it:

- Decisions made, and the reasoning that produced them
- Measurements, benchmarks, and what they ruled in or out
- Gotchas that bit, and what the tell was
- Facts about people, companies, or relationships
- Outcomes another project would need: shipped, abandoned, blocked on someone
- Things learned about how the user works or what they prefer

Ignore the mechanics: which file was edited, which test went red, which command was run.

### Step 3: Apply the rubric

Read `references/rubric.md` and apply it honestly to each candidate. The single test:

> Would future-you, on a different project with a different AI, want this surfacing in
> semantic search?

If the answer is "maybe", the answer is no. Cap the result at **five** candidates. If the
sweep produced more than five, the rubric was applied too loosely — tighten it and rerun the
filter rather than picking the top five arbitrarily.

**If zero candidates survive, say so and stop.** Report what was considered and why it did
not qualify, in one or two lines. Do not lower the bar to produce output.

### Step 4: Write each candidate

Read `references/writing-captures.md`. Each capture is one standalone, self-identifying
statement. It must make sense to an AI that has never seen this session, this repo, or this
conversation.

### Step 5: Dedupe before proposing

For each surviving candidate, call `search_thoughts` with a query drawn from its substance.

- Near-identical hit: drop the candidate and note the existing capture in the report.
- Related but the new one sharpens or contradicts it: keep it, and flag the relationship so
  the user can decide between capturing, superseding, or dropping.
- No hit: keep it.

Skipping this step produces a brain that answers the same question four slightly different
ways. Do not skip it.

### Step 6: Propose for approval

Present the survivors numbered, each showing the exact text to be captured plus one line on
why it passes the test:

```
1. "Recursive graph traversal cost went exponential past hop 3 on a production-scale
    entity graph (27ms at 2 hops, 152s at 6). Cap hop count explicitly rather than
    trusting the planner to stay linear."
    → transfers to any graph traversal, not just this schema

2. ...

Capture all, some (numbers), or none?
```

Wait for the answer. Under `--auto`, skip the wait and capture all survivors.

### Step 7: Capture

For each approved candidate call `capture_thought` with:

- `content` — the standalone statement
- `source_kind` — `"derived"` (a session brief is derived, not a transcript)
- `source_ref` — `claude-code:<PROJECT>@<BRANCH>`
- `participants` — only when real people are named, as `[{"name": "..."}]`. Let the server
  resolve them; no `resolve_entity` pre-call is needed.

Leave `visibility` at its default. Do not pass `predicate_hints` unless the user asked for a
specific claim to be asserted.

If a result carries a `needs_disambiguation` block, surface the question and options to the
user and feed the choice back through `resolve_disambiguation`. Never guess on their behalf
and never leave the block unmentioned.

### Step 8: Report

State what was captured, what was skipped as a duplicate, and what the user declined. Keep it
to a few lines. If nothing was captured, that is a complete and successful report.

## Guidelines

1. **Silence is a valid output.** Most sessions yield nothing durable.
2. **Never capture without approval** unless `--auto` was passed or the user said to go ahead.
3. **One idea per capture.** Two ideas in one statement retrieve badly for both.
4. **Write for a stranger.** No "we", no "the fix", no unexplained pronouns or repo nouns.
5. **Do not restate the repo.** Anything reconstructible by reading HEAD does not belong here.
