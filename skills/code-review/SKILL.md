---
name: code-review
description: Review a branch diff cold for correctness bugs and cleanup opportunities, ranked by confidence, without posting anything. Use when asked to review a diff, a branch, or a PR, and as the fallback when Claude Code's built-in /code-review is not available. Reviews only; it never replies on threads and never commits.
---

# Code review — read the diff cold, rank by confidence

Claude Code ships a built-in `/code-review`. Prefer it when it exists. This skill is the fallback, and
the procedure is the same either way: read the change with no memory of having written it, and report
only what survives a second look.

## 1. Get the diff

```bash
git diff --stat "$(git merge-base HEAD origin/main)"..HEAD
git diff "$(git merge-base HEAD origin/main)"..HEAD
```

For a PR/MR instead of the working branch: `gh pr diff <n>` or `glab mr diff <n>`. Substitute the real
default branch when it is not `main`.

Read the full diff before forming an opinion. Reviewing hunk by hunk is how a reviewer misses that two
correct-looking hunks contradict each other.

## 2. Load the project's own rules

`CLAUDE.md` at the root, plus any `CLAUDE.md` in a directory this diff touched. A violated project
convention is a real finding; a convention the project never stated is your taste, not a finding.

## 3. Review along these passes

Run each pass over the whole diff. They catch different classes, and one pass doing all of them
catches the loud ones only.

- **Correctness** — off-by-one, nil/null, inverted condition, error swallowed, wrong variable, missing
  await, resource never closed, transaction not rolled back. Trace inputs to outputs; do not
  pattern-match on the shape of the code.
- **Blast radius** — grep the callers of every function whose signature or behaviour changed. A change
  that is correct at the call site the diff shows and wrong at the two it does not is the most common
  real bug in review.
- **History** — `git log -p` and `git blame` on the modified lines. Code that looks pointless is
  sometimes a fix for a bug nobody wrote a test for. Removing it reintroduces the bug.
- **Comments and contracts** — read the comments and docstrings in the touched files. A change that
  falsifies a comment either has a bug or has left a lie in the file.
- **Convention** — the project's stated rules, and the surrounding file's idiom.
- **Reuse and simplification** — something already in this codebase does it, the standard library does
  it, an installed dependency does it, or it collapses to fewer lines with no loss.

## 4. Score every candidate before reporting

For each candidate, ask what concrete input or state makes it fail, and what the wrong output is. Then
score confidence 0-100:

- **90-100** — reproduced the failure path in the code, or the rule it violates is written down.
- **70-89** — the reasoning holds but one link is inferred rather than read.
- **40-69** — plausible, unverified. Report as a question, not a defect.
- **below 40** — drop it silently.

Report nothing under 40. A review padded with maybes trains the author to skim it.

## 5. Report

Most severe first. One line each: `path:line — what breaks, and the input that breaks it`. Then the
one-sentence fix. Group cleanups separately from bugs so the author can triage.

No praise, no summary of what the diff does, no style nits that change nothing. If nothing survived
scoring, say that plainly. A clean review is a result.

This skill reports. It does not commit, does not push, and does not post comments on the PR or MR
unless it was explicitly asked to.
