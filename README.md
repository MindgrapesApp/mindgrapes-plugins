# Mind Grapes plugins

Claude Code plugins for [Mind Grapes](https://github.com/MindgrapesApp/mindgrapes-server), a
self-hosted, MCP-accessible second brain.

## Requires a Mind Grapes server

These plugins are clients, not a memory store. They do nothing on their own. You need a
running Mind Grapes server somewhere you can reach, connected to Claude Code as an MCP
server, exposing `capture_thought`, `search_thoughts`, and `resolve_disambiguation`.

Install and run it first: **[MindgrapesApp/mindgrapes-server](https://github.com/MindgrapesApp/mindgrapes-server)**.
It is self-hosted, so "somewhere" can be your laptop, a homelab box, or a small cloud
instance.

## Install

This repository is a plugin marketplace. Add it once, then install the plugins you want.

```
/plugin marketplace add MindgrapesApp/mindgrapes-plugins
/plugin install mindgrapes-session@mindgrapes
```

## Plugins

### `mindgrapes-session`

Adds `/brief`, which sweeps a working session and captures only the parts worth remembering
across projects.

The problem it solves: a memory store that receives every decision becomes a log, and a log
is not worth searching. `/brief` applies one test to everything it finds.

> Would future-you, on a different project with a different AI, want this surfacing in
> semantic search?

Anything that needs the current repo, branch, or conversation to make sense fails that test
and is dropped. Most sessions produce zero or one capture, and reporting zero is a normal
outcome rather than a failure.

How it behaves:

- **Proposes, does not capture.** Candidates are shown with the exact text and the reason
  each one passes. Nothing is written until you approve. `/brief --auto` skips the gate.
- **Dedupes first.** Every candidate is searched against the existing brain, so the same
  fact does not get stored four slightly different ways.
- **Capped at five.** More than five means the filter was applied too loosely.
- **Writes standalone statements.** Each capture is readable by an AI with none of your
  context: no bare pronouns, no unexplained repo nouns.

Usage:

- `/brief` — sweep the current session
- `/brief <hint>` — bias the sweep toward one thread
- `/brief --auto` — capture survivors without the approval step

## Design notes

`/brief` deliberately does not run automatically at session end. A `SessionEnd` hook can only
execute a shell command, so it cannot summarize or reach an interactively-authenticated MCP
server. Driving the sweep from inside a live session avoids both limits.

If automation becomes worthwhile, the intended shape is a `SessionEnd` hook that spools the
transcript path to disk and a `SessionStart` hook that surfaces the pending count, leaving the
capture itself in a real session. That is not built yet, on purpose: the rubric needs to earn
trust manually before anything queues work on its own.

`/brief` also does not replace saying "remember this". A direct request should become a direct
`capture_thought` call with your own words. This skill handles the other case, where you want
the residue of a session without narrating it yourself.

## Relationship to claude-mem

If you run [claude-mem](https://github.com/thedotmack/claude-mem), it already captures what
happened in this repository, automatically. `/brief` deliberately skips that layer. It only
proposes things that would travel to a different project.

## License

MIT
