# Surface New Work

Detect follow-ups that surfaced in the chat but don't map to any existing Linear ID. Propose them as new issues or sub-issues. No writes here — proposals only.

## Quick Decision

1. **Follow-up scoped to a referenced parent's domain?** → propose as **sub-issue** (with `parentId`)
2. **Follow-up is independently shippable?** → propose as **top-level issue**
3. **Near-duplicate exists in the team?** → skip; link in the parent's comment instead

## Inputs

From [gather-context.md](gather-context.md):

- Raw list of chat-surfaced follow-ups
- List of referenced issue IDs (parents available for sub-issues)

From [reconcile.md](reconcile.md):

- Team key (for `list_issues` filter)

## Dedup against existing issues

Before proposing any new issue, fetch a candidate set:

```
mcp__linear-server__list_issues(teamId=<from primary>, limit=100, state=<open states>)
```

For each follow-up, compute a near-duplicate match against existing titles using simple normalization (lowercase, strip punctuation, compare token overlap ≥ 70%). On match: drop the follow-up from the proposal list and note "links to existing `<ID>`" in the parent's comment instead.

## Sub-issue vs top-level

A follow-up is a **sub-issue** when both:

- It only makes sense in the context of a referenced parent (e.g. "add retry logic to the Linear webhook handler" under ABC-17).
- It's small enough that splitting into its own project would be churn.

Otherwise it's a **top-level issue**. When in doubt, prefer top-level — it's easier to reparent later than to extract.

## Proposed issue shape

Each proposal must include:

| Field          | Required   | Notes                                              |
| -------------- | ---------- | -------------------------------------------------- |
| `title`        | yes        | Imperative mood, ≤ 80 chars                        |
| `description`  | yes        | 2–4 sentences, quote the chat source line          |
| `teamId`       | yes        | From the primary referenced issue                  |
| `parentId`     | sub-issues | Linear ID of the parent (no prefix conversion)     |
| `priority`     | optional   | Pull from chat signal (e.g. "high priority bug")   |
| `labels`       | optional   | Match labels on the parent if creating a sub-issue |

## Output of this phase

Two lists:

- `proposedTopLevel: Array<IssueDraft>`
- `proposedSubIssues: Array<IssueDraft & { parentId }>`

Both pass into [writes.md](writes.md) for confirmation + creation.

## What NOT to surface

- TODOs the user explicitly closed in-chat ("never mind, fixed it")
- Refactors with no concrete trigger ("we should clean this up someday")
- Stylistic nitpicks the user dismissed
- Anything already represented by an existing Linear ID

When in doubt, list it — the user gates approval anyway.
