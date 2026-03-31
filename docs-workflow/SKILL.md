---
name: docs-workflow
description: >
  Two-mode documentation skill for a before/after workflow.
  Use when the user says "brief me", "brief-me", "read the docs in",
  "load context from", "update-docs", "update the docs in",
  "write docs after my changes", or "document what I did in".
---

# docs-workflow

Two commands for a before/after documentation workflow.

---

## GitHub CLI rules

These rules apply to both **brief-me** and **update-docs** whenever GitHub issues are involved.

- Use `gh issue view <number>` or `gh issue view <url>` to read issues explicitly referenced by number or URL.
- Use `gh issue list` only when the user explicitly asks to find related issues, or when the repository is known but the issue reference is incomplete.
- Use `gh issue edit` to update issues: supported fields are `--title`, `--body`, `--add-label`, `--remove-label`, `--add-assignee`, `--remove-assignee`, `--milestone`, `--remove-milestone`.
- Use `gh issue close` to close issues: supported flags are `--reason {completed|not planned|duplicate}`, `--comment`, `--duplicate-of`.
- Do not use `--web` on any `gh issue` command.
- Do not run `gh auth login`, `gh auth refresh`, or any other authentication command.
- Do not run a separate availability or authentication check before using `gh`.
- Do not retry a failed `gh` command with alternate auth or fallback flows.
- If a `gh` command fails, report the failure clearly and continue with the remaining non-issue work.

---

## brief-me

**Use when**: the user is about to start work and wants to get up to speed by reading an existing docs folder, optionally alongside referenced GitHub issues.

**Steps**:

1. Read all files in the specified path recursively.
2. If the prompt references GitHub issues by number or URL, use `gh issue view` to read each one. If the prompt asks to find related issues without specifying exact numbers, use `gh issue list` to discover them.
3. If any `gh` command fails, skip the GitHub issue portion and continue with the docs-only summary.
4. Produce a clear, synthesized summary of what the docs and any successfully retrieved GitHub issues cover — what exists, what the key topics are, and how things relate to each other. Don't just list files; explain the content.
5. Highlight any explicit TODOs, open questions, or "future work" items you find in the docs and any successfully retrieved GitHub issues as a quick recap of pending work.
6. Note anything that appears incomplete, missing, or unclear.
7. End with: "Context loaded. Here's what I found — let me know when you're ready to start."

**Output format**:

```
## Summary
...

## Key Topics
- ...

## ⚠️ Gaps or Unclear Areas
- ...

## TODOs and Open Questions
- ...
```

If the path doesn't exist or has no readable files, say so and stop.

---

## update-docs

**Use when**: the user has finished work and wants to create or update docs in a specified folder to reflect what was done.

### Phase 1 — Plan (always run first)

1. Identify what changed this session using the current conversation as the primary source:
   - Infer changes from what the user has described doing (features, tickets, files, refactors, etc.) in this chat
   - If the user or conversation explicitly mentions branches, commits, or asks you to look at git history, then run targeted git commands (for example `git diff`, `git show`, or `git log`) to refine your understanding
   - If the conversation explicitly mentions existing GitHub issues that should be updated or closed, include those as part of the plan
2. Read existing docs in the specified path to understand what's already there
3. Produce a doc update plan — for each proposed change include:
   - Action: `CREATE` or `UPDATE`
   - Target file path
   - One-line reason why
   - For issue updates, use `ISSUE-UPDATE <number>` and specify exactly what will change: `--title`, `--body`, `--add-label`, `--remove-label`, `--add-assignee`, `--remove-assignee`, `--milestone`, or `--remove-milestone`
   - For issue closures, use `ISSUE-CLOSE <number>` and specify the reason (`completed`, `not planned`, or `duplicate`) plus an optional `--comment`

**Plan output format**:

```
## Doc Update Plan

- UPDATE `path/to/existing.md` — reason
- CREATE `path/to/new.md` — reason
- ISSUE-UPDATE `#123` — add-label "done", remove-label "in-progress"
- ISSUE-CLOSE `#456` — reason: completed, comment: "Resolved by docs update"

Proceed with writing? (yes / no)
```

Stop here and wait for user approval before writing anything.

### Phase 2 — Write (only after approval)

4. Execute the approved plan:
   - **Update** existing docs in place, matching their existing tone and structure
   - **Create** new docs, matching the style of other files in the folder
   - Execute only issue actions that appear explicitly in the approved plan — do not infer additional issue work during this phase
   - Use `gh issue edit` for `ISSUE-UPDATE` actions and `gh issue close` for `ISSUE-CLOSE` actions, following the GitHub CLI rules above
5. After writing, list what was created, updated, or changed. For each issue action, report the result as `SUCCEEDED` or `FAILED`.

If you can't determine what changed from the conversation and any explicitly requested git checks, ask the user to describe what they did before producing the plan.
