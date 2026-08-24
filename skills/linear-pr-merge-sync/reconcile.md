# Reconcile Per Issue

For each Linear ID referenced in the PR, draft a Shipped (or Up-for-review) / Still missing / suggested transition update. Read-only — no writes happen here.

## Quick Decision

1. **PR state?** → MERGED uses "Shipped" framing; OPEN uses "Up for review" framing and caps transitions at In Review.
2. **Issue body has explicit acceptance criteria?** → score Shipped against each criterion
3. **Issue body is sparse / one-liner?** → say so, base Still missing only on chat follow-ups
4. **Issue is already in a terminal state (Done / Cancelled)?** → comment only, do not suggest transition

## Extract Linear references

Regex `\b[A-Z]{2,}-\d+\b` across:

- PR title
- PR body
- Branch name (`headRefName`)
- Every commit subject and body
- The chat transcript

De-duplicate. The **primary** issue is the first match in the PR title or body (in that order). Everything else is secondary.

The **team key** is the prefix of the primary ID (e.g. `ABC` from `ABC-17`). Use it when filtering `list_issues` in [new-work.md](new-work.md). (This alpha prefix is only for matching references — the `teamId` that write calls need is the UUID on the fetched issue's `team` field.)

## Fetch state in parallel

For each referenced ID call:

```
mcp__linear-server__get_issue(id, includeChildren=true, includeParent=true)
```

If the primary issue's `project` field is non-null, also call:

```
mcp__linear-server__get_project(id)
```

## Draft per-issue update

Each issue gets a Markdown block with three sections.

### Shipped / Up for review (2–4 bullets)

Source from PR body and commit subjects, **never speculation**. One bullet per concrete deliverable.

- MERGED: header `**Shipped**` — past tense bullets.
- OPEN: header `**Up for review**` — present-tense bullets, referencing the PR.

Example (MERGED):

```
**Shipped**
- Linear OAuth handshake + token refresh (commits abc123, def456)
- Webhook receiver wired to `/api/linear/events`
- Backfill job for existing workspaces
```

Example (OPEN):

```
**Up for review in [PR #42](<url>)**
- Linear OAuth handshake + token refresh (commits abc123, def456)
- Webhook receiver wired to `/api/linear/events`
- Backfill job for existing workspaces
```

### Still missing

Two possible bodies:

- **If issue body has explicit acceptance criteria:** list each unmet criterion verbatim, plus chat-surfaced follow-ups.
- **If issue body is sparse:** open with `_Issue body has no explicit acceptance criteria — basing this only on chat-surfaced follow-ups._` and list those.

If nothing is missing, write `_Nothing — this issue is fully covered by the PR._`

### Suggested status transition

Pick from the rubric below. Surface as a suggestion only — applied only after `AskUserQuestion` approval in [writes.md](writes.md).

**MERGED PR:**

| Coverage                                | Suggestion                       |
| --------------------------------------- | -------------------------------- |
| All criteria met, no follow-ups         | In Progress → Done               |
| Primary slice met, follow-ups deferred  | Stay In Progress (comment only)  |
| Partial / tangential coverage           | Stay In Progress (comment only)  |
| Issue already Done or Cancelled         | No transition — comment only     |

**OPEN PR:**

| Current state                           | Suggestion                       |
| --------------------------------------- | -------------------------------- |
| Backlog / Todo, PR is the first work    | → In Progress                    |
| In Progress, team uses an In Review col | → In Review                      |
| In Progress, no In Review column        | Stay In Progress (comment only)  |
| Issue already Done or Cancelled         | No transition — comment only     |

Never suggest Done on an OPEN PR — wait for the actual merge.

## Output of this phase

A list of `{ issueId, draftComment, suggestedTransition | null }` records, plus the `project` object if applicable. Pass to [writes.md](writes.md).
