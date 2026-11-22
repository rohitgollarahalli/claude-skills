---
name: adversarial-pr
description: Ship a change end to end on a repository you own — branch, test-first change, local checks, push, PR, green CI, two adversarial reviews (/code-review plus an independent reviewer briefed to break the change), fix confirmed findings, re-run CI, merge, log. Use when the user wants a change shipped and merged on their own repo ("ship it", "take this to merge", "do the full PR cycle"). For only writing a commit or PR description use commit-and-pr; for only reviewing a diff use code-reviewer. Never merges on third-party repos.
---

# Adversarial PR cycle

A PR merged on the author's own word is a PR reviewed by the person least able
to see its flaws. This skill makes every merge pass two reviews that are set up
to disagree with the author: an automated diff review and a fresh reviewer who
never sees the author's reasoning.

## Before you start

- Confirm the repo is the user's own and their rules allow agents to merge. If
  not, stop after step 8 (PR open, reviews answered); maintainers merge.
- Find the default branch: `gh repo view --json defaultBranchRef -q
  .defaultBranchRef.name`. Below, `<base>` means that branch.
- Find the allowed merge method (`gh repo view --json
  mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed`) and the project's
  exact build/test/lint commands. Don't guess either.

## The cycle

1. **Branch** from fresh `origin/<base>` (`feat/<slug>`, `fix/<slug>`). Never
   commit to `<base>`. Use a git worktree if the main checkout may hold someone's
   uncommitted work.
2. **Test first.** Write the failing test, then the change. A bug fix starts with
   a regression test that fails on `<base>`.
3. **Local checks** — build, vet, test and lint, all green before pushing. If a
   local *tool* is broken (not the code), you may rely on CI for that step, but
   say so in the PR body.
4. **Push and open the PR.** Conventional Commit title. Body: What / Why / Testing /
   References (cite primary docs for any API, flag or version claim).
5. **Wait for CI** with `gh pr checks <n> --watch`. A red check you didn't cause
   is still yours: reproduce it (stress flaky tests with repeated runs), find the
   cause, and fix it — don't just re-run until green.
6. **Review twice, in parallel:**
   - `/code-review` on the PR.
   - An independent reviewer subagent (a cheaper model is fine) given **only the
     diff and the repo path** — never your conclusions or "I think it's fine".
     Brief it to break the change, and ask it to mark which findings it verified
     by running something.
7. **Triage.** Verify each finding yourself. Fix the confirmed ones (with a test
   where it's behavior). Write down rejected or deferred ones with a one-line reason.
8. **Re-run CI** after fixes. Post a short review summary as a PR comment
   (fixed / deferred / rejected, each with why).
9. **Merge** from outside any worktree that has the branch checked out, using
   the repo's allowed method: `gh pr merge <n> --squash|--merge|--rebase
   --delete-branch -R <owner>/<repo>`. Then `git worktree remove <path>`.
   (`--delete-branch` also cleans up locally and can fail inside a worktree when
   `<base>` is checked out in another one.)
10. **Log** the push, PR and merge (see the `agent-log` skill).

## Why the reviewer gets no context

A reviewer told what the author concluded tends to confirm it. The diff alone
forces it to rebuild intent from the code, which is how real readers and
attackers meet the change.

## Reviewer brief template

```
You are an adversarial reviewer. Break this change; do not approve it.
Repo: <path>, branch <branch>. Diff: `git diff origin/<base>...HEAD`.
Hunt for: edge cases, error paths, security, concurrency, portability, and
docs/changelog that don't match the code. <Add domain-specific risks.>
Verify facts against official docs online. Report findings as: file:line —
severity (blocker/major/minor) — failure scenario — suggested fix —
verified-by-running or reasoned. Do not edit or push.
```

## Example PR comment

```
Review summary (/code-review + independent reviewer)
- Fixed: parser accepted an empty input as valid (regression test added).
- Fixed: CHANGELOG listed a new feature under "Fixed".
- Deferred: pin Actions by SHA — repo-wide change, separate PR.
- Rejected: "add retry" — the caller already retries (see client.go:88).
CI green after fixes.
```

## Guardrails

- Never merge with red CI or an unaddressed blocker.
- Never push to or force-push `<base>`.
- Never claim a check passed that you did not run; report failing output verbatim.
- Tagging a release, publishing, or changing repo settings is out of scope unless
  the user's own rules allow it.
