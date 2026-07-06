# Remnant - AGENTS.md

Canonical Remnant rules for all agents (Codex, Google Antigravity, Cursor, Zed, and any tool that reads `AGENTS.md`). `CLAUDE.md` and `GEMINI.md` point here.

## Required startup

1. Look for `REMNANT.md` in the current project root.
2. If `REMNANT.md` exists, read it before touching project files.
3. If `REMNANT.md` is missing, create it from `REMNANT.template.md`.
4. If both files are missing, create `REMNANT.md` manually using the schema below.
5. Use `REMNANT.md` as a compact context map, not as full chat history.
6. Staleness check: if the `Session` date is older than the latest git commit, trust `git log` over `State` and refresh `State` before relying on it.
7. Read only files needed for the current `## Next` task. Use `## Map` to locate them instead of re-exploring the repo.

## Required shutdown

Before the final response, update `REMNANT.md` with a compact, non-sensitive handoff:

- `History`: roll the previous session's `Done` into one summary line with a keyword tag (e.g. `[auth]`, `[ci]`), append to History (keep last 5, drop oldest)
- `Done`: this session's completed work only — fresh each session, not cumulative
- `Failed`: attempted work that failed and why
- `State`: current project state as a snapshot — replace fully, do not append (max 6 bullets)
- `Next`: exact next task for a new context
- `Blockers`: unresolved decisions or dependencies
- `Decisions`: append any decision made this session as `date — decision — why`. Never rewrite or drop existing entries; this section is the project's durable memory.
- `Map`: add or correct key files touched this session, one line each (`path — role`). Keep it a map, not an inventory.

Keep the whole file under ~150 lines. If a section pushes past that, compress the ephemeral sections (`Done`, `State`), never `Decisions`.

## Compression rule

Apply when: session exceeds 30 messages, context is approaching its limit, or the user requests it.

Steps:
1. Summarize decisions — what was decided and why, not the full discussion. Append durable ones to `## Decisions`.
2. List changed files — paths only, no diffs or content. Update `## Map` for key files.
3. Capture current state — what works, what is broken, what is in progress.
4. Write the exact next step — specific enough for a new agent to start without reading the conversation.
5. Record blockers — unresolved questions, decisions needed, or dependencies.

Never store secrets, credentials, tokens, private chat text, personal data, raw terminal output, or full conversation logs in `REMNANT.md`.
Never commit `REMNANT.md`; it is local-only and must stay in `.gitignore`.
Never use Remnant CLI commands; Remnant is Markdown-only.

## REMNANT.md schema

Version 2. Ephemeral sections (`Session` through `Blockers`) rotate each session. Durable sections (`Decisions`, `Map`) only grow or get corrected — never rolled away.

```markdown
<!-- AGENT PROTOCOL: read this file fully before touching code.
     Before your final response: update Done/Failed/State/Next/Blockers,
     roll Done into History (keep 5), append decisions to Decisions.
     Never commit this file. -->
# Remnant - <project-name>

## Session
- version: 2
- date: <ISO 8601>
- agent: <claude-code | codex | gemini-cli | antigravity | other>
- duration: <minutes>

## History
- <[tag] one-line summary per previous session — oldest first, max 5 — or "None">

## Done
- <completed work — this session only>

## Failed
- <failed attempt and reason — or "None">

## State
- <current state as a snapshot — replace each session, max 6 bullets>

## Next
- <exact next step>

## Blockers
- <open question or dependency — or "None">

## Decisions
- <YYYY-MM-DD — decision — why. Append-only, never compressed>

## Map
- <path — one-line role of this key file>
```

A file without `version:` or without `Decisions`/`Map` is schema v1: migrate it by adding the missing sections, keep all existing content.

## Project shape

```text
AGENTS.md            canonical agent rules (this file)
CLAUDE.md            Claude Code pointer to AGENTS.md
GEMINI.md            Gemini CLI pointer to AGENTS.md
REMNANT.template.md  safe template to commit
REMNANT.md           local-only memory, ignored by Git
```
