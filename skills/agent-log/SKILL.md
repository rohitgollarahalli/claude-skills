---
name: agent-log
description: Keep a durable, append-only change log of what an agent did so any fresh session can resume from files alone. Use when the user or project keeps an agent change log (e.g. `~/AGENT_LOG.md`) or asks for one — read its tail at session start and append an entry after every push, PR, merge, release, skill install or settings change. Also use for "what did you do", "where did we leave off", or "log this". Do not create a log file the user never asked for.
---

# Agent change log

Conversations get cleared and compacted; a log file does not. The log is the
single place a new session reads to learn the last change and what is still open,
so it must be written the moment something happens, not reconstructed later.

## Location

Use the path the user's rules name. `~/AGENT_LOG.md` is the usual default. If
no log exists and nobody asked for one, don't create it — ask once or skip.
Newest entry at the bottom.

## At session start

1. Read the tail (`tail -60 <log>`).
2. Group entries by item (same repo and PR or branch). **Only the newest entry
   for each item counts** — the log is append-only, so a 13:15 "waiting on CI"
   entry is superseded by a 14:20 "merged, done" entry for the same PR.
3. Resume items whose newest entry has a `Status` other than `done`, or a `Next`
   other than `none`.
4. If the tail looks mid-story (an item with no closing entry), `grep` the log
   for that item before assuming it is still open.

## When to append

After each of: push, PR opened, PR merged, release tagged, skill or plugin
installed, settings or timer changed, something blocked. One entry per event or
per tightly related group.

## Entry format

```
## YYYY-MM-DD HH:MM — <repo or area> — <short title>
- Action: branch/push/PR opened/merged/skill installed/…
- Branch / PR: <name> / <URL>
- What & why: <1–2 lines>
- Status: done | waiting on CI | waiting on maintainer | blocked: <reason>
- Next: <what to do next, or "none">
```

`Status` uses exactly one of those values, so a later reader can filter on it.
Put anything else (e.g. "and then review") in `Next`. Take the time from
`date '+%F %H:%M'`; never guess it.

## Rules

- **Append only.** Never edit or delete an old entry. Correct a mistake with a new
  entry that says what was wrong.
- **Facts, not intentions.** "PR #14 opened", not "will open PR". Plans go in `Next`.
- **Real URLs** copied from command output, never constructed by hand.
- **Blocked means say why** and what would unblock it (e.g. "token lacks
  `workflow` scope; user runs `gh auth refresh -s workflow`").
- **Who does it.** If a step belongs to the human (tagging a release, approving
  text), say so in `Next` ("user tags v0.2.0") so a later agent doesn't do it.
- Append with a heredoc (`cat >> <log> <<'EOF'`) so existing content is never
  rewritten.

## Example

```
## 2026-11-22 13:15 — mcpx — v0.2.0 release PR opened
- Action: pushed feat/release-v0.2.0, opened PR
- Branch / PR: feat/release-v0.2.0 / https://github.com/owner/mcpx/pull/14
- What & why: first release (binaries, image, install.sh) so users can install without Go.
- Status: waiting on CI
- Next: adversarial review, fix findings, merge; then user tags v0.2.0
```

## Guardrails

- Never log secrets, tokens, or private customer data.
- Never mark `done` what was not verified done.
- At a real boundary (PR merged, topic change), write the entry first, then tell
  the user it is a good moment to clear context.
