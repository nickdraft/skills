# Gather Context

Read everything before drafting anything. All steps here are read-only.

## Quick Decision

1. **PR number known?** → use `gh pr view <num> --json …`
2. **Only branch known?** → `gh pr view --json …` from the current checkout
3. **No PR at all?** → stop and ask the user; this skill is PR-scoped

## Resolve the PR

```bash
gh pr view <num> --json number,title,body,state,isDraft,mergedAt,baseRefName,headRefName,url,commits,author
```

Capture:

- `state` — `OPEN` or `MERGED` (also `CLOSED` — treat as "stop and ask").
- `isDraft` — if true, stop and ask before continuing.
- `mergedAt` — present only when `state == MERGED`; used in comment header.
- `url` — included in every Linear comment downstream.

State drives downstream behavior:

- **OPEN** — frame the comment as "Up for review", cap transitions at In Review, and treat commit list as "proposed to ship".
- **MERGED** — frame the comment as "Shipped", Done transitions are allowed (still gated on approval).

## Read commit history

```bash
git log <baseRefName>..<headRefName> --format='%H%x09%s%n%b%n--END--'
```

Extract:

- Commit subjects (the "what shipped / what's up for review" bullets pull from these).
- Commit bodies (often contain `ABC-XX` references that the title omits).
- Commit SHAs (cite in every Linear comment).

If the PR was squash-merged, prefer the merge commit's body — squashing collapses subjects into the merge commit, and the original commit history may be unreachable post-merge. For OPEN PRs, the commit history on the head branch is authoritative.

## Read diff scope (not contents)

```bash
git diff <base>...<head> --stat
```

Use only for **scope signals**: file count, directories touched. Do NOT line-by-line review the diff — that's the `review` skill's job.

## Scan the chat transcript

Re-read the current session for follow-up signals:

| Signal                                       | Treat as                            |
| -------------------------------------------- | ----------------------------------- |
| "TODO", "FIXME" added in code                | Untracked follow-up                 |
| "we'll do X later", "out of scope"           | Untracked follow-up                 |
| User caught a bug but said "ship anyway"     | Untracked follow-up (high priority) |
| User explicitly said "this is in ABC-99"     | Already-tracked, link in comment    |
| Tests skipped or marked `.todo`              | Untracked follow-up                 |
| Refactor deferred                            | Untracked follow-up (low priority)  |

## Output of this phase

A context bundle with:

- PR metadata (number, url, title, body, state, isDraft, mergedAt-or-null)
- List of commit SHAs + subjects
- Files-touched summary (count + top dirs)
- Raw list of Linear-ID-shaped strings found anywhere
- Raw list of chat-surfaced follow-ups (one line each, with source quote)

Pass this bundle into [reconcile.md](reconcile.md) and [new-work.md](new-work.md).
