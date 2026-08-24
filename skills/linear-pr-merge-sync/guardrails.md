# Guardrails

Safety rules every phase of the skill must respect. If a rule conflicts with a quick path, the rule wins.

## Hard rules

| Rule                                              | Why                                                       |
| ------------------------------------------------- | --------------------------------------------------------- |
| No write before matching `AskUserQuestion`        | The user owns the Linear trail — no surprise edits        |
| Capture `state` (OPEN/MERGED) before running      | Comment wording and transition options depend on state    |
| Never suggest Done on an OPEN PR                  | The PR hasn't landed; Done would mislead readers          |
| Never auto-transition status                      | Status changes notify humans; user must approve each      |
| Never invent acceptance criteria                  | Honest "criteria sparse" beats fabricated checklists      |
| Always pre-check `list_issues` before creating    | Duplicates pollute the tracker and erode trust            |
| Always include PR URL + commit SHAs in comments   | Future readers must be able to audit the trail            |
| Detect team key from data — never hardcode prefix | Skill must work in any repo, not just `ABC`               |

## Soft rules

- Prefer top-level issue over sub-issue when ambiguous; reparenting is cheap, extraction is not.
- Prefer `atRisk` over `offTrack` when project health is unclear.
- Prefer "stay In Progress + comment" over "transition to In Review/Done" when coverage is partial.
- For open PRs, prefer comment-only over status transition unless the issue is clearly moving forward (e.g. Backlog → In Progress when the PR is the first concrete work).
- Quote the chat source line in every new-issue description so the proposer's intent survives.

## When to stop and ask instead of guessing

| Situation                                          | Action                                              |
| -------------------------------------------------- | --------------------------------------------------- |
| PR is a draft                                      | Stop, ask: skip / run anyway / wait for ready       |
| No Linear references found anywhere                | Stop, ask: skip / file new / point at an ID         |
| Multiple primary candidates (title + body differ)  | Ask the user which is primary                       |
| Issue is in a terminal state (Done / Cancelled)    | Comment only, do not suggest transition             |
| Project has no recent status update                | Default to skip Q4 unless user requests             |
| Linear MCP returns an error mid-write              | Stop, surface the error, do not retry blindly       |

## Failure modes to watch for

| Failure                                              | Mitigation                                            |
| ---------------------------------------------------- | ----------------------------------------------------- |
| UI-button merge with no chat message                 | Run at PR creation; re-run after merge if user pings  |
| Squash-merge collapses commit history                | Read merge commit body for original subjects          |
| Branch names with no Linear ID                       | Fall back to PR title / body / chat references        |
| Two PRs reference the same issue back-to-back        | Comment idempotently — don't assume previous state    |
| Issue moved to a different team mid-flight           | Re-fetch the issue before writing                     |
| Long chat with stale follow-ups                      | Filter to follow-ups raised after the PR was opened   |
| User changes their mind mid-confirmation             | Skip that category, continue with others              |

## Don't

- Don't run on individual commits.
- Don't run on draft PRs without explicit confirmation.
- Don't suggest a Done transition on an OPEN PR — cap at In Review.
- Don't post a single "approve everything" prompt.
- Don't summarize what the skill did at the end of the run — the final report from [writes.md](writes.md) is the summary.
- Don't edit existing Linear comments — always post new ones; the trail is append-only by design.
