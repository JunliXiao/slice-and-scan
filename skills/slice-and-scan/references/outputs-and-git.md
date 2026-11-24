# Outputs and Git Reference (`outputs-and-git.md`)

How to handle derived outputs (`CSV`, reports, split files) and generated scripts relative to local workspaces, `.gitignore`, version control (such as Git or GitLab), and recurring workflows on legacy or active codebases.

## 1. Keep Derived Outputs Out of Git

Outputs produced by `slice`, `scan`, or `stat` scripts are **derived artifacts**—intermediate views, reports, or summaries built for inspection, impact scoping, or AI reasoning—not source code or canonical documentation.

* **Stale data is worse than no data:** On active or legacy repositories being debugged, refactored, or extended, a committed report or table quickly drifts out of sync with the codebase and can mislead engineers or AI into reasoning on outdated facts.
* **Avoid merge conflicts:** When multiple people edit entities, mappings, routes, or workflows in parallel branches, shared summary files become an unnecessary source of merge friction.
* **Keep search and context clean:** Committing large derived tables or split files adds noise to repository searches, diffs, and agent file lookups.

### Practical Rule

Put derived outputs in a dedicated local folder (such as `.slice-scan-out/` or an explicit output path) and ignore it in `.gitignore`:

```gitignore
.slice-scan-out/
```

*(Exception: a frozen, read-only archival/archaeology snapshot where the codebase itself is no longer changing and an explicit point-in-time record is intentionally desired.)*

## 2. Disposable Scripts vs. Promoting Recurring Scripts

Not every generated script deserves to live permanently in a repository:

* **Default to disposable:** A one-off script written to investigate a specific bug, check one refactor risk, or profile a single export can stay in a temporary/working area and be discarded once the task is finished. Keeping one-off scripts out of the repo avoids clutter and cognitive overhead.
* **Promote when recurring:** If a script addresses a high-frequency need in that module—such as mapping entity/table associations, auditing raw statement/data-access patterns, or checking recurring workflow references—and its conventions prove reliable across more than one task, promote a clean version into a dedicated folder in the repo (for example, `tools/slice-and-scan/`) with brief usage notes.

## 3. Why Clean Scripts Help Both Humans and AI

When a legacy codebase has accumulated historical conventions or mixed patterns, a well-kept recurring script in `tools/slice-and-scan/` serves two practical roles:

* **Fresh, accurate snapshots on demand:** Any engineer can run it locally when needed to inspect the current state of the branch rather than relying on a stale committed file.
* **Living convention context for AI:** When generating a new script for the same repository later, the agent can inspect existing scripts in `tools/slice-and-scan/` to pick up local conventions, path habits, and edge-case handling already proven in that codebase.

## 4. CI & Guardrails (Only When Worth It)

During active refactoring or architecture cleanup, recurring scripts can optionally be wired into CI (such as GitLab CI):

* **On-demand or artifact reports:** Run on demand or on branches/schedules to attach fresh summaries as job artifacts without committing generated files into Git history.
* **Architecture checks:** Use a check script as a lightweight guardrail when enforcing a specific migration rule—only when the team actively maintains that rule so it does not become noisy CI overhead.