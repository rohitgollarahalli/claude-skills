---
name: adversarial-pr
description: Run the full own-repo pull-request cycle with an adversarial review before merge — branch, test-first change, local checks, push, PR, green CI, /code-review plus an independent reviewer briefed to break the change, fix confirmed findings, re-run CI, merge, log. Use whenever you are about to open, review, or merge a PR on a repo you own, or when the user says "ship it", "open a PR", "merge this", or "review this before merging". Fires even if the user does not name the skill.
---

# Adversarial PR cycle

A PR merged on the author's own word is a PR reviewed by the person least able
to see its flaws. This skill makes every merge pass two reviews that are set up
to disagree with the author: an automated diff review and a fresh reviewer who
never sees the author's reasoning.

## The cycle

1. **Branch** from fresh `origin/main` (`feat/<slug>`, `fix/<slug>`). Never commit
   to `main`. Use a worktree if the main checkout may hold someone's uncommitted work.
2. **Test first.** Write the failing test, then the change. A bug fix starts with
   a regression test that fails on `main`.
3. **Local checks** — the project's exact build, vet, test and lint commands (look
   them up; do not guess). All green before pushing. If a local tool is broken
   (not the code), say so in the PR body and rely on CI for that step.
4. **Push and open the PR.** Conventional Commit title. Body: What / Why / Testing /
   References (cite primary docs for any API, flag or version claim).
5. **Wait for CI** with `gh pr checks <n> --watch`. Fix until green.
6. **Review twice, in parallel:**
   - `/code-review` on the PR.
   - An independent reviewer subagent (a cheaper model is fine) given **only the
     diff and the repo path** — never your conclusions or "I think it's fine".
     Brief it to break the change: edge cases, security, concurrency, error paths,
     portability, docs and changelog that don't match the code. Ask it to mark which
     findings it verified by running something.
7. **Triage.** For each finding, verify it yourself. Fix the confirmed ones (with a
   test where it's behavior). Note rejected ones with a one-line reason.
8. **Re-run CI** after fixes. Post a short review summary as a PR comment
   (found / fixed / rejected-and-why).
9. **Merge** with `gh pr merge <n> --merge --delete-branch`. Remove the worktree.
10. **Log** the push, PR and merge (see the `agent-log` skill).

## Why the reviewer gets no context

A reviewer told what the author concluded tends to confirm it. The diff alone
forces it to rebuild intent from the code, which is how real readers and
attackers meet the change.

## Reviewer brief template

```
You are an adversarial reviewer. Break this change; do not approve it.
Repo: <path>, branch <branch>. Diff: `git diff origin/main...HEAD`.
Hunt for: <domain-specific risks>. Verify facts against official docs online.
Report findings as: file:line — severity (blocker/major/minor) — failure
scenario — suggested fix — verified-by-running? Do not edit or push.
```

## Example PR comment

```
Review summary (code-review + independent reviewer)
- Fixed: install.sh treated an empty checksum match as success (added test).
- Fixed: CHANGELOG listed a feature under "Fixed".
- Rejected: "pin actions by SHA" — repo policy pins by major tag; tracked separately.
CI green after fixes.
```

## Guardrails

- Never merge with red CI or an unaddressed blocker.
- Never push to or force-push `main`.
- Never claim a check passed that you did not run; report failing output verbatim.
- On third-party repos stop after opening the PR and answering review comments —
  maintainers merge.
- Tagging a release, publishing, or changing repo settings is out of scope unless
  the user's own rules allow it.
