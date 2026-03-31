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

## brief-me

**Use when**: the user is about to start work and wants to get up to speed by reading an existing docs folder, optionally alongside referenced GitHub issues.

If the prompt also specifies GitHub issues, use the GitHub CLI to read them:

- Use `gh issue list` to discover or confirm relevant issues when needed
- Use `gh issue view` to read each specified issue in detail
- Do not run a separate availability or authentication check first
- Do not run `gh auth login` or any fallback authentication flow
- If a `gh` command fails for any reason, skip the GitHub issue portion and continue with the docs-only summary

**Steps**:

1. Read all files in the specified path recursively.
2. If the prompt specifies GitHub issues, attempt to gather the relevant issue context with `gh issue list` and `gh issue view`, without running a separate availability or authentication check first.
3. If any `gh` command fails, skip the GitHub issue portion and continue with the docs-only summary.
4. Produce a clear, synthesized summary of what the docs and any successfully retrieved GitHub issues cover — what exists, what the key topics are, and how things relate to each other. Don't just list files; explain the content.
5. Highlight any explicit TODOs, open questions, or "future work" items you find in the docs and any successfully retrieved GitHub issues as a quick recap of pending work
6. Note anything that appears incomplete, missing, or unclear
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
   - If GitHub issue work is needed, include issue actions using `ISSUE-UPDATE` or `ISSUE-CLOSE`, the target issue number or URL, and a one-line reason why

**Plan output format**:

```
## Doc Update Plan

- UPDATE `path/to/existing.md` — reason
- CREATE `path/to/new.md` — reason
- ISSUE-UPDATE `#123` — reason
- ISSUE-CLOSE `#456` — reason

Proceed with writing? (yes / no)
```

Stop here and wait for user approval before writing anything.

### Phase 2 — Write (only after approval)

4. Execute the approved plan:
   - **Update** existing docs in place, matching their existing tone and structure
   - **Create** new docs, matching the style of other files in the folder
   - If the approved plan includes GitHub issue work, use `gh issue edit` to update existing issues and `gh issue close` to close them
   - Do not run a separate availability or authentication check first
   - Do not run `gh auth login` or any fallback authentication flow
   - If a `gh` command fails, report the failure clearly and continue with the remaining approved doc work
5. After writing, list what was created, updated, or changed in GitHub issues.

If you can't determine what changed from the conversation and any explicitly requested git checks, ask the user to describe what they did before producing the plan.
