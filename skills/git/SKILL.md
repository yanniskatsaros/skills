---
name: git
description: Guidance for using git, the `gh` CLI, and opening pull requests (PR's)
---

- Do not commit any changes or open PR's unless you are explicitly asked to do so
- Use conventional commits
- Commit changes grouped by a logical, atomic change or unit of work
- Infer if the branch contains a ticket (ex: `yannis/fri-1234-some-feature-name`) if available, and prefix the PR title with it: "[FRI-1234] ..."
- Descriptions for PR's must:
  - Remain concise, and easy to scan by a reader.
  - Use simple, plain English and avoid jargon.
  - Use bullet points to describe the changes following this pattern:

  ```
  - {summary of change}
    - **what**: {description of what changed}
    - **why**: {description of why we did it}
  ```
