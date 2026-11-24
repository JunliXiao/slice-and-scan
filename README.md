# slice-and-scan

An agent skill that generates small, repeatable, non-interactive scripts on demand to compress large files, cross-file relationships, or tabular data into compact tables or statistical summaries before reasoning.

Rather than dumping bulky raw files into the model's context window, `slice-and-scan` lets the agent generate a focused script in your chosen language and local runtime—such as single-file Java, Python, PowerShell, or other tools already in your environment—for legacy maintenance, debugging, refactoring, performance profiling, and impact assessment. You can run just one step (`slice`, `scan`, or `stat`) or chain them together as needed, keeping lightweight, derived files (`CSV`, reports, or split outputs) for repeated inspection and AI analysis.

The agent adapts each generated script to your environment and task without dragging in heavy tooling.

Why this approach fits agentic workflows:

* **Saves tokens:** Compresses large codebases, workflow definitions, or bulky data exports into small, high-signal tables or statistical summaries before deep analysis.

* **Keeps context and scope clean:** Separates deterministic file/text processing (CPU) from architectural judgment and trade-off analysis (LLM), staying focused on read-only observation rather than modifying or executing existing code, attaching to live processes, or simulating full compilers/control flow.

* **Puts both ends on the same row:** Resolves cross-file links, annotations, or field relationships in code and writes related items together on the same row or record, so the agent does not have to match distant pieces across a long context.

* **Reports what it missed, not just what it found:** Every script reports its scanned, matched, skipped, and unresolved counts, so a zero result is never mistaken for proof of absence.

* **Matches a lightweight, composable workflow:** Supports standalone steps or chained runs (`slice` -> `scan` / `stat` -> save derived output) without forcing rigid pipelines or heavy frameworks.

* **Works at three levels of autonomy:** The agent can run a script and act on its output, you and the agent can work through it together, or you can take the script and run it by hand as you always have. Scripts print a plain console summary, so the same output serves the agent, a collaborator, and a person at a terminal.

## Philosophy

Begin by looking. Before building any instrument, read a few things with your own eyes: a handful of files, a short sample of rows, until the shape of the terrain is familiar. 

When the terrain is small, or the answer lives in subtle intent rather than in volume, looking is the whole job, and a script would only be ceremony. 

When an authoritative source already knows the answer (a schema export, a dependency graph, a profiler), ask it, and use your own scripts only for the gaps it leaves.

Reach for a script when the question outgrows the eye: when there is more than can be held in view, or when two distant things must be seen side by side. Then build the smallest instrument that can answer this one question, no more general than the question requires and honest about what it cannot see. Aim it at a small piece first. Read what it returns the way you would read a witness: with interest, and with the habit of checking its account against the source.

Hold the instrument loosely. Refine it when the evidence demands, set it down when the question is answered, and keep it only if the same question keeps coming back. Tools exist to serve understanding, and understanding is what you carry forward.

## Install

Copy `skills/slice-and-scan/` into your agent's skills directory (for example, workspace `.agents/skills/slice-and-scan/` or your agent's global skills folder). Works with agents supporting [Agent Skills](https://agentskills.io) (such as Gemini CLI / Antigravity CLI, Claude Code, Codex, Cursor, OpenCode). Then ask for what you want to slice, scan, or compute stats on, and specify your preferred language/runtime if desired:

* "use `slice-and-scan` to split a large PL/SQL package or LOB/DEL file into per-procedure or per-package files so I can inspect specific routines"

* "scan our JPA entities and join `@Table` schema/name metadata across files into a CSV"

* "scan for raw `Statement` / `createStatement` calls and map each hit to its class, method, and line"

* "slice/normalize `NAME` values into token and full-string views, then compute `STRING_LENGTH` and `TOKEN_COUNT` distributions across Chinese, English, and mixed groups (`PR50`, `PR95`, `PR99`)"

* "trace recursive `.kjb` and `.ktr` workflow references and flag missing files or circular references before a batch change"

## How It Works: Slice, Scan, Stat

Rather than locking you into a rigid pipeline, `slice-and-scan` treats slicing, scanning, and statistical analysis as composable modes you can use alone or together:

1. **Slice：** Breaks large monolithic files or bulky records into manageable pieces, scoped extracts, or normalized intermediate views (for example, splitting large SQL/LOB/DEL files by `PROCEDURE`, `FUNCTION`, or `CREATE OR REPLACE PACKAGE`, or deriving token/full-string views from delimited fields).

2. **Scan：** Batch-scans source code, XML/workflow files, or configuration to trace cross-file mappings, distant associations (such as annotations, entity/table links, or workflow references), and pattern/risk locations with context metadata.

3. **Stat：** Groups records and computes summary metrics and percentiles (`MIN`, `PR25`, `PR50`, `PR75`, `PR95`, `PR99`, `MAX`) to verify invariants, understand data/string/token shapes, and guide performance or capacity decisions.

### Scope, Modules & Version Control

* **Single-module-first & multi-repo discipline:** Designed primarily around a single module or dataset at a time. For changes spanning multiple repositories or heterogeneous stacks, prefer **one focused script per module/repo** outputting compact summaries, correlated afterward if needed, rather than one bloated cross-repo script.

* **Read-only observation boundaries:** Built for non-invasive inspection, structural discovery, invariant profiling, and impact checks—not writing code/data, attaching to live processes/JVMs, or simulating full compilers/ASTs/control flow.

* **Derived outputs & Git hygiene:** Derived outputs (`CSV`, summaries, split files) are meant for repeated observation and reasoning, not committing as stale snapshots into Git/GitLab (put output folders such as `.slice-scan-out/` in `.gitignore`). Keep one-off scripts disposable, and promote stable, high-frequency scripts (for example under `tools/slice-and-scan/`) only when repeatedly useful across tasks or CI checks.

## What is inside

All files in `skills/slice-and-scan/` are pure Markdown documents—no pre-built script files or runtime dependencies are bundled:

```
skills/slice-and-scan/
|-- SKILL.md                 routing, pre-flight checks, slice/scan/stat workflow, safety boundaries, output rules
|-- references/              loaded on demand:
|   |-- modes-and-flow.md    slice, scan, and stat modes—standalone or chained—and when to use each
|   |-- outputs-and-git.md   derived files, .gitignore rules, disposable vs. promoted scripts
|   `-- multi-module.md      single-module-first habit and per-repo outputs for cross-module impact checks
|-- templates/               language-agnostic structural blueprints for generating scripts on demand:
|   |-- slice-template.md    stream boundary splitting, scoped extraction, and field normalization pattern
|   |-- scan-template.md     index-and-resolve pattern for cross-file, annotation, and reference scans
|   `-- stat-template.md     grouped streaming and percentile/distribution summary pattern
`-- examples/                concrete problem-to-script walkthroughs from real tasks:
    |-- association-scan.md  cross-file entity/table lookup and compact CSV export
    |-- feature-scan.md      comment/string stripping, class/method tracking, and categorized findings
    |-- file-slice.md        splitting large SQL/LOB/DEL streams into scoped units
    `-- quantile-stat.md     grouping text/records and computing min/percentile distributions

```

## License

MIT

## Credits

Regulus Technologies