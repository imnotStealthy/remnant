# Remnant

![Remnant logo](assets/remnant-logo.png)

No install. No server. No CLI. Just Markdown.

Remnant is a session memory protocol for AI coding agents. One ignored file, `REMNANT.md`, lets Claude Code, Codex, Gemini CLI, Antigravity — or any LLM that can read a file — resume a session without asking you to re-explain the project.

## How It Works

The agent reads `REMNANT.md` at startup and writes a compact handoff before ending:

```text
REMNANT.md
|-- Session    date, agent, duration, schema version
|-- History    one-line summaries of previous sessions (max 5)
|-- Done       completed work this session
|-- Failed     failed attempts and reason
|-- State      current project state (snapshot, replaced each session)
|-- Next       exact next step
|-- Blockers   unresolved questions or dependencies
|-- Decisions  durable: date — decision — why (append-only)
`-- Map        durable: key files and their roles
```

Two kinds of memory:

- **Ephemeral** (`Session` → `Blockers`) — the handoff. Rotates each session; old `Done` rolls into `History`.
- **Durable** (`Decisions`, `Map`) — the project's long-term memory. Decisions and the file map survive every session and never get compressed away.

The file starts with an embedded agent protocol comment, so it is **self-describing**: any LLM that opens it — even one with no Remnant instruction file — learns the read/update rules from the file itself.

## Setup

**1. Copy the instruction files into your project root:**

| File | Read by |
|------|---------|
| `AGENTS.md` | Codex, Antigravity, Cursor, Zed — canonical rules |
| `CLAUDE.md` | Claude Code (pointer to `AGENTS.md`) |
| `GEMINI.md` | Gemini CLI (pointer to `AGENTS.md`) |

`AGENTS.md` holds the full rules; the others are two-line pointers, so the rules can never drift between agents. Tools that read none of these still work via the protocol banner inside `REMNANT.md`.

**2. Copy `REMNANT.template.md` into your project root.**

**3. Add `REMNANT.md` to `.gitignore`:**

```gitignore
REMNANT.md
```

`REMNANT.md` is created automatically the first time the agent runs.

## Starting a Session

Tell the agent once at the start of a new project:

```text
Use Remnant. If REMNANT.md is missing, create it from REMNANT.template.md.
Read it before touching files. Before ending, update it with the final handoff.
```

After that, the instruction files handle startup and shutdown automatically.

## When Context Is Getting Full

If a session runs long or the context window is filling up, ask the agent to compress:

```text
Compress the session into REMNANT.md: decisions, changed files, current state, exact next step, blockers.
```

The agent writes a summary, not a transcript. It captures only what a new agent needs to continue.

## Agent Rules

Full rules live in `AGENTS.md`. Summary:

### Startup

1. Look for `REMNANT.md` in the project root.
2. If it exists, read it before touching any files.
3. If it is missing, create it from `REMNANT.template.md`.
4. If the `Session` date is older than the latest git commit, trust `git log` over `State`.
5. Use `Map` to locate key files instead of re-exploring the repo; read only files needed for `## Next`.

### Shutdown

Before the final response, update `REMNANT.md`:

```text
History    roll previous Done into one tagged line (keep last 5)
Done       this session's completed work only — fresh each session
Failed     failed attempts and reason
State      snapshot, max 6 bullets — replace, do not append
Next       exact next task for a new context
Blockers   unresolved decisions or dependencies
Decisions  append "date — decision — why" — never rewrite or drop
Map        add/correct key files touched this session
```

Keep the whole file under ~150 lines; compress ephemeral sections first, never `Decisions`.

## Example

A handoff after multiple sessions:

```markdown
<!-- AGENT PROTOCOL: read this file fully before touching code.
     Before your final response: update Done/Failed/State/Next/Blockers,
     roll Done into History (keep 5), append decisions to Decisions.
     Never commit this file. -->
# Remnant - remnant

## Session
- version: 2
- date: 2026-05-05T14:00:00Z
- agent: claude-code
- duration: 30

## History
- [pivot] 2026-05-04 (codex, 90m): repositioned as Markdown-only protocol, removed CLI

## Done
- Improved compression rule in agent files.
- Restructured README for beginner clarity.

## Failed
- None.

## State
- All agent files consistent.
- README restructured, zero-install preserved.

## Next
- Review and publish.

## Blockers
- None.

## Decisions
- 2026-05-04 — Markdown-only, no CLI — zero-install is the core value proposition.

## Map
- AGENTS.md — canonical agent rules
- README.md — user-facing docs
```

## Schema

See the full schema in `AGENTS.md` or `REMNANT.template.md`. A `REMNANT.md` without `version:` or without `Decisions`/`Map` is schema v1; agents migrate it by adding the missing sections and keeping all content.

## Security

`REMNANT.md` is local-only. Keep it in `.gitignore`.

Write:
- decisions and rationale
- changed file paths
- current project state
- exact next action
- blockers

Never write:
- secrets, API keys, credentials
- private chat logs or personal data
- raw terminal output or build logs
- irrelevant history

## Files

```text
AGENTS.md            canonical agent rules
CLAUDE.md            Claude Code pointer to AGENTS.md
GEMINI.md            Gemini CLI pointer to AGENTS.md
REMNANT.template.md  safe template to commit
REMNANT.md           local-only memory, ignored by Git
```
