---
name: linear-pr-merge-sync
description: "Reconcile Linear issues, sub-issues, and project status updates against a PR — comments what shipped (or what's up for review), flags what's still missing, and files new issues/sub-issues for follow-up work surfaced in the chat. Use this skill when the user creates a PR (e.g. 'open PR', 'gh pr create', 'send this for review', 'ready for review') OR merges a PR ('merge PRs', 'merge these', 'land all open PRs'), says 'sync Linear', 'update Linear from this PR', 'close the ticket', 'reconcile Linear', or asks what's left. Some tools merge a PR via a UI button with no chat message (e.g. the GitHub merge button), so the reliable hook is PR creation — run the skill then, and optionally re-run after merge. Run it once per PR that references a Linear ticket (ABC-/etc. in title, body, or branch name). Triggers on: PR creation, opening a PR, gh pr create, send for review, ready for review, PR merge, batch PR merge, merging multiple PRs, post-merge cleanup, Linear sync, Linear update, ticket close, ticket reconcile, sub-issue, project status update, follow-up issue, what's still missing, what shipped, ABC-, gh pr merge, mergedAt, UI merge button."
---

# Linear PR Sync

> **Requires:** a Linear MCP server (`mcp__linear-server__*` tools).

A reconciliation workflow for closing the loop between a PR (open or merged) and the Linear issues, sub-issues, and project status updates that should reflect it.

## Quick Reference

| Phase                                         | When to Use                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| [Gather context](gather-context.md)           | Resolve the PR, read commits/diff, scan the chat transcript  |
| [Reconcile per issue](reconcile.md)           | Draft Shipped / Still missing / suggested status transition  |
| [Surface new work](new-work.md)               | Propose new issues or sub-issues from untracked follow-ups   |
| [Apply writes](writes.md)                     | Comment, transition, create issue, post project status       |
| [Guardrails](guardrails.md)                   | Safety rules every step must respect                         |

## Core Principles

### 1. PR Creation or Merge

Trigger on PR creation OR PR merge. Both are valid entry points:

- **OPEN** — the PR was just created and is up for review. Comment with "Up for review", suggest at most an In Progress → In Review style transition. Never auto-close.
- **MERGED** — the PR has landed. Comment with "Shipped", and Done transitions are on the table (still gated on approval).

With some tools, merging happens via a UI button and produces no chat message, so PR creation is usually the last reliable chat hook. Re-running the skill after merge is safe — comments are append-only.

Never run on an individual commit, or a draft PR, without explicit user confirmation.

### 2. Read-Only Until Approved

No `mcp__linear-server__save_*` call before the user approves the matching `AskUserQuestion`. One question per category, never a single "approve everything" prompt.

### 3. Detect, Don't Hardcode

The Linear team key is whatever issue prefix appears first in the PR (e.g. `ABC-`, `XYZ-`). Detecting from data keeps the skill reusable across repos.

### 4. Honest Gaps

Never invent acceptance criteria. If the issue body is sparse, say so explicitly and base "Still missing" only on what surfaced in the chat.

### 5. Auditable Trail

Every Linear comment includes the PR URL and the list of commit SHAs that landed (or are proposed) under it. Future readers should be able to walk from issue → PR → diff in one click.

## Decision Flowcharts

### Should I Transition Status?

```
Is the PR MERGED?
├── No (OPEN) → never suggest Done. At most suggest In Progress → In Review.
│              If issue is still in Backlog/Todo, suggest → In Progress.
└── Yes (MERGED)
    └── Did the PR cover every acceptance criterion in the issue?
        ├── Yes → Suggest In Progress → Done (ask before applying)
        └── No
            ├── Did it cover the primary slice with follow-ups deferred?
            │   └── Yes → Stay In Progress, post comment with "Still missing"
            └── Was the work tangential / partial?
                └── Yes → Stay In Progress, do not suggest transition
```

### New Top-Level Issue or Sub-Issue?

```
Does the follow-up depend on a referenced parent's domain?
├── Yes → Sub-issue (pass parentId)
└── No
    ├── Is it independently shippable?
    │   └── Yes → Top-level issue
    └── Is it a known-existing issue?
        └── Yes → Skip — link in comment instead
```

## Common Mistakes

| Mistake                                       | Fix                                                     |
| --------------------------------------------- | ------------------------------------------------------- |
| Auto-transitioning status without asking      | Always gate transitions on `AskUserQuestion`            |
| Suggesting Done on an open PR                 | Cap open-PR transitions at In Review                    |
| Hardcoding `ABC-` as the team prefix          | Detect from the first reference in the PR               |
| Speculating acceptance criteria               | Quote the issue body verbatim or say it's sparse        |
| Filing a duplicate issue                      | Pre-check `list_issues` before proposing new ones       |
| Comment without PR URL or commit SHAs         | Always include both for audit                           |
| Running on a draft PR silently                | Verify state, confirm with user if `isDraft`            |
| One mega "approve all" prompt                 | One `AskUserQuestion` per write category                |
| Posting `offTrack` whenever anything is open  | Use the health rubric in [writes.md](writes.md)         |

## Review Checklist

Before reporting completion:

- [ ] PR state captured (OPEN or MERGED) and used to gate transitions
- [ ] Linear references extracted from PR title, body, commits, branch, chat
- [ ] Each referenced issue fetched and its state inspected
- [ ] Per-issue Shipped / Up-for-review / Still missing draft prepared
- [ ] Untracked follow-ups surfaced and dedup-checked against existing issues
- [ ] Each write category gated on a separate `AskUserQuestion`
- [ ] Comments include PR URL + commit SHAs
- [ ] Status transitions only after explicit approval; Done only on MERGED
- [ ] Final report lists every Linear URL touched and every new ID created

## Reference Files

- [gather-context.md](gather-context.md) — PR resolution, commit/diff parsing, chat-transcript scan patterns
- [reconcile.md](reconcile.md) — Drafting Shipped / Still missing / suggested transition per issue
- [new-work.md](new-work.md) — Surfacing untracked follow-ups, sub-issue vs top-level, dedup
- [writes.md](writes.md) — Linear MCP write APIs, comment template, project status health rubric
- [guardrails.md](guardrails.md) — Safety rules and common failure modes
