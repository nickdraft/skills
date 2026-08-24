---
name: linear-prioritise-issues
description: Establish a clear execution order for a set of Linear issues by wiring a blocking chain and adjusting priorities so the sequence reads at a glance in list views. Use when the user wants to "prioritise issues in order", "set the order for milestone X", "wire a blocking chain", "make the sequence clear in Linear", sequence a phase/milestone, or order a backlog deterministically. Operates on an explicit issue list or a project+milestone filter. Only wires hard blockers; surfaces soft deps as caveats.
---

# Prioritise Linear issues

> **Requires:** a Linear MCP server (`mcp__linear-server__*` tools).

Wires an explicit execution order across a set of Linear issues using **blocking relations** + **priority bumps**, so anyone scanning the milestone in Linear sees the sequence immediately.

## When to invoke

- "Prioritise these issues in order: A → B → C"
- "Set the order for milestone X / Phase N"
- "Wire a blocking chain for these tickets"
- "Make the Phase N (or milestone) sequence clear in Linear"

## Inputs

- **Issue set**: either an explicit ordered list (`ABC-1, ABC-2, ABC-3`) OR a `project + milestone` filter (then ask user for desired order).
- **Order**: required. If absent, propose one from dependency analysis and confirm before writing.

## Workflow

1. **Load the issues.** Use `mcp__linear-server__list_issues` with `project` + `milestone` (or `mcp__linear-server__get_issue` per ID). Capture: id, title, description, priority, parent, existing `blocks`/`blockedBy`, milestone.

2. **Sanity-check the order against descriptions.** Scan each description for dependency cues:
   - Explicit: "blocked by", "depends on", "requires", "after X lands".
   - Structural: "extends ABC-X", "builds on", parent/child links.
   - Logical: B reads data that A produces (e.g. the report step reads data the import step writes).

   If the proposed order conflicts with a stated dependency, **stop and flag it** before writing. Do not silently re-order.

3. **Classify dependencies.**
   - **Hard** (same milestone, must-be-done-before): wire as blocker.
   - **Soft** (cross-milestone, "could ship without", description-only mention): do NOT wire. Surface in the summary.
   - **Cross-phase/cross-milestone**: never auto-wire; flag as caveat unless user explicitly asks.

4. **Preview, then confirm.** Render the planned blocking chain and the priority diff (current → proposed) for the user, and get explicit confirmation before issuing any `save_issue` write.

5. **Apply the chain.** For each adjacent pair `(A, B)` in the order, call:
   ```
   mcp__linear-server__save_issue({ id: "A", blocks: ["B"] })
   ```
   Apply `blocks` additively; confirm your Linear MCP's write call preserves existing relations (append) rather than replacing them before running across a milestone. Use `blockedBy` only when wiring the inverse direction is clearer (rare).

6. **Adjust priorities so the chain is visible in list view.** Default scheme (Linear priority ints: 0 none · 1 urgent · 2 high · 3 medium · 4 low):
   - First 1–2 issues → **High (2)**
   - Middle issues → **Medium (3)**
   - Last issue → keep existing, or **Low (4)** if it was unset

   **Never** auto-set Urgent (1). **Never** downgrade a priority that's already higher than the scheme would assign. If an issue is already Urgent, leave it.

7. **Summarise.** Output:
   - The resulting chain with issue links
   - Priorities set (and any left alone)
   - Soft/cross-milestone deps flagged as caveats
   - One sentence inviting the user to escalate any soft dep to a hard blocker

## Worked example

A data-pipeline milestone had three issues. Order requested: ABC-1 → ABC-2 → ABC-3.

- ABC-3's description says "Extends ABC-2 …" → ABC-2 must precede ABC-3 (hard, structural).
- ABC-2 (import/transform step) reads data that ABC-1 (schema/definition step) produces → ABC-1 must precede ABC-2 (hard, logical).
- ABC-3's export path depends on a later milestone's work → **soft, cross-milestone** — flag, don't wire.

Writes:
- `save_issue(ABC-1, blocks=["ABC-2"], priority=2)`
- `save_issue(ABC-2, blocks=["ABC-3"], priority=2)`
- ABC-3 left at Medium.

Caveat surfaced: "ABC-3's export depends on the next milestone (ABC-4/ABC-5). Left as a soft dep — say the word if you want it hard-wired."

## Don'ts

- Don't wire cross-milestone blockers without explicit user OK.
- Don't auto-set Urgent.
- Don't downgrade an existing higher priority.
- Don't reorder silently when a description contradicts the requested order — flag first.
- Don't remove existing blockers (apply `blocks` additively and confirm your Linear MCP's write call appends rather than replaces existing relations; use `removeBlocks` only on explicit request).
