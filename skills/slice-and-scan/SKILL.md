---
name: slice-and-scan
version: 1.0.0
description: >-
  Generates small, repeatable, non-interactive scripts on demand to slice
  large files or fields, scan code/config/workflows for patterns and
  cross-file relationships, or compute grouped statistical distributions
  into compact derived files before reasoning. Use when inspecting legacy
  codebases, tracing cross-file associations or references, profiling
  data/text lengths or token counts, or doing pre-change impact checks
  without dumping bulky raw files into context.
---

# slice-and-scan

Generate focused, non-interactive local scripts on demand that compress large files, cross-file relationships, or tabular datasets into compact tables, split outputs, or statistical summaries for human inspection and AI reasoning.

This skill consists solely of pure Markdown instructions, blueprints, reference notes, and examples—no bundled script binaries or external runtime dependencies. Adapt every generated script to the user's requested language, local runtime/environment, and task scope.

## Core Principles

1. **Compress before reasoning:** Avoid loading hundreds of raw source files or massive data exports directly into context when a small script can extract relationships, locate matches, or summarize distributions into a compact file first.  
2. **Put both ends on the same row:** When tracing cross-file links, annotations, or distant references, join the related sides onto the same row or record so reasoning does not have to stitch distant context together.  
3. **Composable, not rigid:** Support standalone `slice`, `scan`, or `stat` steps as well as chained runs (`slice` \-\> `scan` / `stat` \-\> save derived output), matching whatever rhythm the task calls for.  
4. **Stay strictly read-only and non-invasive:** Generate observation, slicing, scanning, and profiling scripts—never modify business logic in place, attach to live processes/JVMs, or claim full compiler/AST/control-flow simulation from heuristic text parsing.  
5. **Single-module-first, derived outputs out of Git:** Focus on one module or dataset at a time, and keep scripts and derived outputs in the gitignored sandbox described under Workspace & File Layout.
6. **Report what was missed, not just what was found:** Every script must report total scanned, matched, skipped, and unresolved counts, and these must reconcile: the parts add up to the whole. Before drawing conclusions, spot-check 2–3 random rows from the output against the original source.

## Pre-Flight Check

Before generating a script, check whether a script is actually warranted and which mode fits:

* **Direct read vs. script:** If the target is a handful of small files (\< \~30 files) or a tiny sample, reading directly or doing a quick search may be cheaper than writing and running a script. If there are many files, large exports, or cross-file joins, generating a script pays off.  
* **Existing exact tools vs. custom script:** If an exact source of truth already exists (such as exporting database DDL directly or using a dedicated profiler), note that rather than over-claiming heuristic text/regex parsing.  
* **Sample first when unfamiliar:** On an unfamiliar legacy codebase or format, sample a few files or rows first to confirm conventions (comments, delimiters, annotations, path styles) before scanning or slicing the whole set.  
* **Choose mode(s):**  
  * **Slice** (`templates/slice-template.md`): Split large/monolithic files into units, extract scoped subsets, or normalize/derive fields into intermediate outputs.  
  * **Scan** (`templates/scan-template.md`): Batch-scan files or definitions to find patterns/risks, classify conventions, or trace cross-file/recursive references.  
  * **Stat** (`templates/stat-template.md`): Group records/values and compute counts, min/max, and percentiles (`PR25`, `PR50`, `PR75`, `PR95`, `PR99`) to check invariants or capacity/performance shapes.

## Standard Workflow

1. **Clarify target, mode, and runtime:** 
   * If the project already has a `tools/slice-and-scan/` folder, look there first for scripts that can be reused or adapted, and follow the conventions they already use.
   * Identify the input path(s), desired output (`slice`, `scan`, `stat`, or a sequence), and preferred language/runtime (defaulting to whatever lightweight runtime fits the user's environment and task, such as single-file Java, Python, or PowerShell). If any critical detail is ambiguous, state a reasonable assumption and proceed.  
2. **Load the matching blueprint and references on demand:**  
   * Read the corresponding template in `templates/` (`slice-template.md`, `scan-template.md`, `stat-template.md`) before writing a script.  
   * Load only the needed file(s) under `references/` or `examples/` rather than reading everything at once.  
3. **Generate a clean, self-contained script:** Keep the script focused, readable, and non-interactive, with safe path/error handling, clear CLI arguments or documented entry points, and a concise console summary (including the counts required by Principle 6) plus saved derived output.
4. **Run and verify when appropriate:** 
   * If executing in the user's environment is requested or helpful, run the script on a small or target input, verify that it runs cleanly and produces non-empty expected output, and apply Principle 6.  
   * If the agent cannot run it, or the data is large, sensitive, or on another machine, give the user the exact command and ask them to paste back the console summary (counts and reconciliation) plus 2–3 sample rows from the output, then continue from that.
5. **Reason over the derived output:** Read the derived output directly when it is small (up to a few hundred lines). Large outputs are not meant to be read: peek at the header and a few rows only to learn the columns, then analyze them with the next script (filter, aggregate, or join) and reason over that script's compact result.

## Workspace & File Layout

Keep exploratory work isolated from version-controlled assets using two standard paths:

```
<project-root>/
├── .slice-scan-out/            # Gitignored sandbox for local exploration
│   ├── scripts/                # Newly generated and iterating scripts (e.g., scan_jpa.java); developers decide when a script is promoted
│   └── <derived-outputs>       # Reports, CSVs, and split files produced by scripts
└── tools/slice-and-scan/       # Version-controlled, proven scripts kept by developers
```

* **Write to `.slice-scan-out/`:** Place generated/iterating scripts in `.slice-scan-out/scripts/` and derived outputs in `.slice-scan-out/` (gitignored) so exploratory runs stay isolated from source trees and version control.
* **Exclude `.slice-scan-out/` from traversal:** Scripts must skip this folder when scanning the project, so their own scripts and outputs never show up as findings.
* **Check the ignore rule:** If `.slice-scan-out/` is not in the project's `.gitignore`, tell the user to add it rather than editing the file yourself.

## On-Demand Loading Guide

Load files only when relevant to the current task:

* `templates/slice-template.md` — Structural blueprint for splitting files, extracting subsets, or deriving/normalizing fields.  
* `templates/scan-template.md` — Structural blueprint for batch pattern scans, classification, and cross-file/recursive reference resolution.  
* `templates/stat-template.md` — Structural blueprint for grouped aggregation and percentile/distribution analysis.  
* `references/modes-and-flow.md` — How `slice`, `scan`, and `stat` work standalone or chained.  
* `references/outputs-and-git.md` — Derived output conventions, `.gitignore` rules, disposable scripts vs. promoting recurring scripts into a repo.  
* `references/multi-module.md` — Single-module-first habit and per-module/per-repo scripts \+ summary correlation for cross-module changes.  
* `examples/association-scan.md` — Cross-file entity/table association scanning and compact CSV output.  
* `examples/feature-scan.md` — Comment/string stripping, scope/method tracking, and categorized scan reporting.  
* `examples/file-slice.md` — Splitting large SQL/LOB/DEL streams into scoped units.  
* `examples/quantile-stat.md` — Text/record categorization and percentile distribution summaries.

## Guardrails

* **Never touch live processes or runtime instrumentation:** Do not generate process-attach, JVM agent injection, or live-runtime mutation scripts under this skill.  
* **Never modify target source/data in place as a side effect:** Write derived outputs to a separate output path/folder (such as `.slice-scan-out/` or an explicit output path).  
* **State heuristic limits honestly:** Regex, line/block splitting, and lightweight text/XML parsing can miss dynamic dispatch, inheritance/XML overrides, or unusual formatting. Surface caveats clearly so results are used as a fast, reliable map rather than an infallible proof.