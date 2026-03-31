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

**Use when**: the user is about to start work and wants to get up to speed by reading an existing docs folder.

**Steps**:

1. Read all files in the specified path (recursively)
2. Produce a clear, synthesized summary of what the docs cover — what exists, what the key topics are, and how things relate to each other. Don't just list files; explain the content.
3. Highlight any explicit TODOs, open questions, or "future work" items you find in the docs as a quick recap of pending work
4. Note anything that appears incomplete, missing, or unclear
5. End with: "Context loaded. Here's what I found — let me know when you're ready to start."

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
2. Read existing docs in the specified path to understand what's already there
3. Produce a doc update plan — for each proposed change include:
   - Action: `CREATE` or `UPDATE`
   - Target file path
   - One-line reason why

**Plan output format**:

```
## Doc Update Plan

- UPDATE `path/to/existing.md` — reason
- CREATE `path/to/new.md` — reason

Proceed with writing? (yes / no)
```

Stop here and wait for user approval before writing anything.

### Phase 2 — Write (only after approval)

4. Execute the approved plan:
   - **Update** existing docs in place, matching their existing tone and structure
   - **Create** new docs, matching the style of other files in the folder
5. After writing, list what was created or updated.

If you can't determine what changed from the conversation and any explicitly requested git checks, ask the user to describe what they did before producing the plan.
