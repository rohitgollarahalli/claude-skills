---
name: agent-log
description: Keep a durable, append-only change log of what an agent did so any fresh session can resume from files alone. Use at session start (read the tail), and after every push, PR, merge, release, skill install, or settings change (append an entry). Also use when the user asks "what did you do", "where did we leave off", or "log this". Fires even if the user does not name the skill.
---

# Agent change log

Conversations get cleared and compacted; a log file does not. The log is the
single place a new session reads to learn the last change and what is still open,
so it must be written the moment something happens, not reconstructed later.

## Location

`~/AGENT_LOG.md` unless the project or user rules name another file. Newest entry
at the bottom.

## At session start

Read the last ~30–40 lines (`tail -40 ~/AGENT_LOG.md`) before doing anything else.
Pick up any entry whose `Status` is not `done` or whose `Next` is not `none`.

## When to append

After each of: push, PR opened, PR merged, release tagged, skill or plugin
installed, settings or timer changed, something blocked. One entry per event or
per tightly-related group (e.g. "PR opened" and "merged" can be separate entries
an hour apart — that is the point).

## Entry format

```
## YYYY-MM-DD HH:MM — <repo or area> — <short title>
- Action: branch/push/PR opened/merged/skill installed/…
- Branch / PR: <name> / <URL>
- What & why: <1–2 lines>
- Status: done | waiting on CI | waiting on maintainer | blocked: <reason>
- Next: <what to do next, or "none">
```

Use local time from `date '+%F %H:%M'`; never guess the time.

## Rules

- **Append only.** Never edit or delete an old entry. Correct a mistake with a new
  entry that says what was wrong.
- **Facts, not intentions.** "PR #14 opened" — not "will open PR". Put plans in `Next`.
- **Real URLs** copied from command output, never constructed by hand.
- **Blocked means say why** and what would unblock it (e.g. "token lacks
  `workflow` scope; user runs `gh auth refresh -s workflow`").
- Append with a heredoc (`cat >> ~/AGENT_LOG.md <<'EOF'`) so existing content is
  never rewritten.

## Example

```
## 2026-11-22 13:15 — mcpx — v0.2.0 release PR opened
- Action: pushed feat/release-v0.2.0, opened PR
- Branch / PR: feat/release-v0.2.0 / https://github.com/owner/mcpx/pull/14
- What & why: first release (binaries, image, install.sh) so users can install without Go.
- Status: waiting on CI + adversarial review
- Next: fix findings, merge, tag v0.2.0
```

## Guardrails

- Never log secrets, tokens, or private customer data.
- Never mark `done` what was not verified done.
- At a real boundary (PR merged, topic change), write the entry first, then tell
  the user it is a good moment to clear context.
