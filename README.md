# Agent Skills

A personal collection of agent skills for GitHub Copilot.

## Skills

- **docs-workflow** — Three-command documentation skill. `brief-me` loads context from a docs folder (also triggers on "catch me up", "summarize the docs", etc.). `update-docs` creates or updates docs after work is done. `update-standards` manages code standards files (code examples allowed). Append `with issues` to `brief-me` or `update-docs` to include GitHub issue work.

  ```bash
  npx skills@latest add GoldMyr1994/skills/docs-workflow
  ```
