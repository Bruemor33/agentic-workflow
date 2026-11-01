---
name: vallen-code-review
description: Review the committed changes on a git branch against its base using only local git, with no GitHub/GitLab/PR integration. Use when the user asks to review a branch, review what was pushed, or code-review the current work.
---

# Vallen Code Review (local git only)

Review the committed diff of a branch. Everything comes from `git`; never call a PR API, never post comments anywhere, never modify the branch. Findings are written to the docs root, not the repo.

## 1. Resolve the target

- **Branch**: the one the user named, else the current `HEAD` of this session. If `HEAD` is detached, ask.
- **Base**: the one the user named, else the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main` then `master`).
- Run `git fetch origin <branch> <base>` so you review what is actually pushed. If the fetch fails (offline, no auth), say so and continue with local refs.
- Prefer `origin/<branch>` when it exists; otherwise the local branch. State which ref and which commit SHAs you reviewed.
- Review **committed** code only. If the working tree is dirty or the local branch is ahead of `origin/<branch>`, mention it in one line and leave those changes out.
- Diff against the merge base: `git diff <base-ref>...<branch-ref>`. Get the file list with `--stat` and `--name-status` first. If the diff is huge (roughly >2,000 changed lines), tell the user and propose splitting by directory or commit range before continuing.
- Read files from the reviewed ref (`git show <ref>:<path>`) or a temporary `git worktree add` of it, never by checking the branch out over the user's work. Remove any temporary worktree when done.

## 2. Gather context

Before judging the diff, learn what it is meant to do: `git log <base>..<branch>` messages, the branch name, and any ticket or spec the user points to (read from the destinations config's tracker/docs root if asked). Read the surrounding code of every changed hunk, the callers of changed functions, and the existing tests, not just the diff lines. Note the project's conventions (CLAUDE.md, lint config, neighbouring code) so you flag deviations from them rather than your own taste.

## 3. Review

Spawn parallel subagents, one per lens, each given the diff, the base/branch refs and the intent summary, and each told to report only findings it can ground in specific lines:

1. **Correctness**: logic errors, off-by-one, null/undefined and error paths, race conditions, wrong assumptions about callers, broken invariants, changed behaviour not reflected in tests.
2. **Security**: injection, authz/authn gaps, secrets or tokens in code, unsafe deserialization, SSRF/path traversal, PII in logs, new dependencies and what they pull in.
3. **Tests and contracts**: missing or weak tests for new behaviour, API/schema/migration compatibility, config and env changes that need rollout steps.
4. **Maintainability**: duplication of existing helpers, needless complexity, dead code, naming that misleads, deviations from project conventions. Lowest priority; keep it short.

For a small diff (a few files), do all four lenses yourself instead of spawning.

## 4. Verify before reporting

For every candidate finding, re-read the code and try to disprove it: is it handled elsewhere, guarded by a caller, impossible given types or data? Drop what you cannot ground. Mark each survivor `confirmed` (you traced a concrete failing input or path) or `plausible` (looks wrong, depends on something you could not see; say what). Do not pad: zero findings is a valid result.

## 5. Report

Write `<docs_root>/reviews/<branch-slug>-<short-sha>.md` (docs root from the `setup-destinations` config; run that skill first if there is none). Never write it into the repo. Format:

```markdown
# Review: <branch> → <base>
Reviewed: <branch-ref> @ <sha> (merge base <sha>), <N files, +A/-D>
Intent: <one or two sentences>
Not reviewed: <uncommitted/unpushed changes, skipped generated files, anything unreadable>

## Must fix
### 1. <short title> — `path/to/file.ext:LINE` — confirmed|plausible
<what is wrong, the concrete scenario that breaks, suggested fix>

## Should fix
## Nits
## Questions for the author
```

Order by severity. Every finding cites `path:line` on the reviewed ref. Omit empty sections. Then tell the user the file path and summarise the Must-fix items in chat in a few lines.

## Limits

- This is a read-only reviewer: do not apply fixes unless the user then asks, and do not push, comment on, or open anything on a remote.
- Skip generated, vendored and lockfile content except to check that new dependencies are expected.
- Say plainly what you could not verify (no ability to run the code, missing test infrastructure), and do run the project's tests or linter from a temporary worktree only if the user wants that.
