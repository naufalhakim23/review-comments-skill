# review-comments

A Claude Code skill for handling review feedback on a GitLab MR or a GitHub PR.

A review bot leaves comments. Some are already fixed, some are real, some are stale or wrong. This
skill decides which is which by reading the current code, fixes what is real, runs a cold `/code-review`
pass on top, cleans up the comments in the touched files, then commits, pushes, and replies on the
threads.

## Install

```
/plugin marketplace add naufalhakim23/review-comments-skill
/plugin install review-comments@review-comments-marketplace
```

## Use

Paste an MR/PR URL with review feedback on it, or say "check the MR comments" / "handle the PR review".
With no URL, the skill derives the target from the current branch.

## Flow

1. Resolve the target (URL, or the MR/PR for the current branch)
2. Pull every thread, paginated, dropping system notes and resolved threads
3. Verify each comment against the code on the current branch: already handled / valid / not valid
4. Fix what is real, root cause over symptom, with every new check seen failing before the fix lands
5. Run `code-review` over the same branch, before committing, and fix what it confirms
6. Clean up the comments in every file the branch touched
7. Commit
8. Push, then reply on each thread citing the pushed SHA
9. Report grouped by verdict

Replies come after the push. A reply citing a SHA nobody can open is worse than no reply.
Resolving threads stays a human decision.

Stacked MRs are handled explicitly: when a chained branch gets a finding on a file its parent owns, the
fix is committed on the parent, then merged forward, so both MRs carry it.

## Requirements

- `glab` authenticated for GitLab, `gh` authenticated for GitHub
- `python3` for the thread-parsing snippets

## Bundled skills

- `review-comments` — the main flow above
- `code-review` — a standalone cold-diff review skill, used by step 5. Claude Code's built-in
  `/code-review` takes precedence when it is installed; this one is the fallback so the flow works
  without it. It reports findings and never commits or posts them.

## License

MIT
