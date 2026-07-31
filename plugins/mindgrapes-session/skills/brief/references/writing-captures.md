# Writing a capture

A capture is read back by an AI that has none of your context. Not this session, not this
repo, not this conversation. Write for that reader.

## The standalone test

Read the statement cold, with nothing around it. If any noun is unexplained or any pronoun
has no referent, rewrite it.

Bad:

> We decided to cap it at 3 because of the cliff we found.

Nothing in that sentence identifies itself. Cap what, at 3 what, which cliff, who is "we".

Good:

> Recursive entity-graph traversal cost went exponential past hop 3 on a production-scale
> database (27ms at 2 hops, 152 seconds at 6), so hop count should be capped explicitly
> rather than left to the query planner.

## Rules

1. **Name the subject.** "Mind Grapes' entity graph", not "the graph". "Postgres recursive
   CTEs", not "the query".
2. **One idea per capture.** Two ideas in one statement retrieve badly for both. Split them.
3. **Include the evidence.** The measurement, the error, the observed behaviour. A claim
   without evidence reads as an assertion later and cannot be re-evaluated.
4. **State it as a durable fact, not as news.** Not "today we discovered X". Just X, plus how
   it came to be known.
5. **Keep dates out of the prose** unless the timing is the point. The capture is timestamped
   server-side. "As of July 2026" only earns its place when the fact is expected to expire.
6. **No markdown structure.** Prose sentences. Headers and bullet lists inside a capture
   fragment badly under embedding.
7. **Two to four sentences.** Long enough to stand alone, short enough to stay one idea.

## People

When a real person is named, pass them in `participants` rather than relying on the name
appearing in the text:

```json
{
  "content": "...",
  "participants": [{"name": "<full name as used in the session>"}],
  "source_kind": "derived",
  "source_ref": "claude-code:<project>@<branch>"
}
```

The server resolves each name itself: reuse, provisional bind, or create. A `resolve_entity`
pre-call is optional and usually unnecessary.

Watch for `needs_disambiguation` in the result. It means the server made a best guess and
wants it confirmed. Surface the question and the options to the user, then feed the answer
through `resolve_disambiguation`. An unconfirmed guess also lands in `review_queue`, so
ignoring it does not lose the information, it just leaves a chore behind.

## Provenance

- `source_kind: "derived"` — a session brief is a summary, not a transcript
- `source_ref: "claude-code:<project>@<branch>"` — enough to trace it back

Provenance is metadata, not a substitute for a self-contained statement. A capture that only
makes sense once you look up its `source_ref` has failed the standalone test.

## Worked examples

Session produced a benchmark:

> Copying a production-scale database as a standing benchmark target caught a read-path cost
> cliff that a unit-test-sized fixture could not have shown. Read-path changes should be
> validated against realistic data volume before merge.

Session produced a fact about a person (name it in `participants`, and in the text, exactly as
the session used it):

> [Name] owns the vendor relationship for the payments integration and prefers being asked in
> writing before a call is scheduled.

Session produced a reversal:

> An earlier plan to denormalize survivor entity IDs onto edges was deferred because the
> flatten-on-merge approach solved the same traversal problem without a schema change.
> Revisit only if merge chains reappear at depth.

Session produced nothing:

Capture nothing. Report that the session was implementation work already covered by
claude-mem, and stop.
