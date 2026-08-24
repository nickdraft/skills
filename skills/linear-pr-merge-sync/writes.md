# Apply Writes

Every write in this phase is gated on a user approval via `AskUserQuestion`. One question per category — no mega-prompts.

## Quick Decision

1. **Posting comments?** → ask once, list all targets
2. **Transitioning status?** → ask once per issue (transitions are individually meaningful)
3. **Creating new issues / sub-issues?** → ask once, list all proposals
4. **Posting project status?** → ask once, only if a project is linked

## Approval prompts

```
Q1: Post these comments on <ID1>, <ID2>, ...?  [yes / edit / skip]
Q2: Transition <ID> from <currentState> to <suggestedState>?  [yes / no]   (one Q2 per transition)
Q3: Create N new issues and M new sub-issues?  [review list / yes / skip]
Q4: Post a status update on project "<name>"?  [yes / no]
```

Skip Q4 entirely if no project is linked. For OPEN PRs, Q4 is usually still skipped unless the user explicitly wants a project ping pre-merge.

## Comment template

Every comment posted via `mcp__linear-server__save_comment` follows one of two shapes depending on PR state.

**MERGED:**

```markdown
**Shipped in [PR #<num>](<PR url>)** (merged <mergedAt>)

<Shipped bullets from reconcile.md>

**Still missing**

<Still missing block from reconcile.md>

---
Commits: <SHA1>, <SHA2>, <SHA3>
```

**OPEN:**

```markdown
**Up for review in [PR #<num>](<PR url>)** (opened by @<author>)

<Up-for-review bullets from reconcile.md>

**Still missing**

<Still missing block from reconcile.md>

---
Commits on branch: <SHA1>, <SHA2>, <SHA3>
```

Every comment must include the PR URL and the commit SHA list. Future readers should be able to walk from comment → PR → diff in one click.

## Status transitions

Use `mcp__linear-server__save_issue` with the issue's existing `id` and the new `stateId`. Look up the target state from the team's available statuses via `mcp__linear-server__list_issue_statuses` (cache once per session).

For OPEN PRs, valid targets are at most In Progress / In Review (whichever the team uses). Never apply Done from this skill until the PR is MERGED.

## Creating issues / sub-issues

Use `mcp__linear-server__save_issue` (no `id`). Required fields:

- `teamId` — from the primary referenced issue
- `title`, `description` — from the proposal
- `parentId` — only for sub-issues

Optional pass-through: `priority`, `labelIds`, `projectId` (inherit parent's project if creating a sub-issue).

After each create, capture the returned `identifier` (e.g. `ABC-218`) and `url` for the final report.

## Project status update

Only fires if Q4 was approved. Use `mcp__linear-server__save_status_update`:

```
{
  projectId,
  body: <markdown summary: PR shipped/up-for-review, issues touched, follow-ups filed>,
  health: <see rubric>
}
```

### Health rubric

Apply only on MERGED runs. On OPEN PRs, skip the status update by default — wait until the PR lands so health reflects shipped reality.

| Condition                                                                          | Health      |
| ---------------------------------------------------------------------------------- | ----------- |
| Every referenced issue is now Done; no follow-ups filed                            | `onTrack`   |
| Some follow-ups filed, but no referenced issue is materially incomplete            | `onTrack`   |
| Referenced issue stayed In Progress with non-trivial Still missing                 | `atRisk`    |
| A referenced issue is blocked, missing major scope, or follow-ups outnumber wins   | `offTrack`  |

When in doubt, prefer `atRisk` over `offTrack` — it's easier to upgrade than to walk back alarm.

## Final report

After all approved writes complete, print:

```
Linear sync complete (<PR state>):
  - Commented: ABC-17 (https://...), ABC-32 (https://...)
  - Transitioned: ABC-17 → In Review
  - Created: ABC-218 (sub-issue of ABC-17), ABC-219
  - Project status: skipped (PR still open)
```

Nothing else. No trailing summary of what the skill did — the report is the summary.
