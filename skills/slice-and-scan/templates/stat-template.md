# Stat Blueprint (`stat-template.md`)

A language-agnostic structural blueprint for generating **Stat** scripts—scripts that group, filter, or normalize records/fields and compute summary counts and distribution metrics (such as min, max, and percentiles) to inspect data shapes, verify invariants, and inform performance or capacity decisions.

Use this blueprint when generating a script for a **Stat** request. Adapt syntax, parsing rules, grouping logic, and output format to the user's requested language, runtime, and data format.

## 1. When to Use This Blueprint

Use a **Stat** script when you want quantitative summary and distribution signals over many rows or values rather than reading raw rows or guessing from a tiny sample, especially when:

* **Profiling lengths, counts, or numeric distributions:** Measuring distributions (such as `MIN`, `PR25`, `PR50`, `PR75`, `PR95`, `PR99`, `MAX`) across string lengths, token/segment counts, payload sizes, or numeric fields.
* **Comparing groups or categories:** Breaking metrics down across segments (such as character/script categories like Chinese, English, or mixed text, or types/statuses/groups) to see where heavy tails or anomalies live.
* **Checking invariants or estimating capacity/performance limits:** Establishing empirical baselines and tail behavior before tuning queries, buffers, token budgets, schemas, or processing limits.

## 2. Core Pipeline Structure

A well-formed stat script follows five stages, keeping memory bounded and outputs easy to inspect:

1. **CLI, Input & Metric Setup:** Accept one or more input files (or slice/derived files), optional output file/folder, optional grouping/metric options, and optional row limits for quick checks. Fail fast if required inputs or target columns are missing.
2. **Header & Record Parsing:** Read headers dynamically (matching column names case-insensitively/trimmed rather than hardcoding fragile column indices), parse rows cleanly (handling quotes, escaped quotes, and configured delimiters such as `,` or `;`), and skip blank lines safely.
3. **Grouping & Metric Extraction:** Classify or group each record into its target bucket(s) and extract numeric values or compute derived metrics (such as length or token/segment count) safely without crashing on malformed or non-numeric cells.
4. **Sorting & Percentile/Summary Computation:** Sort collected numeric series per group and compute summary statistics (`COUNT`, `MIN`, percentiles, `MAX`, or requested aggregates) using a consistent, documented method.
5. **Report Writing & Console Summary:** Emit a compact summary table (`CSV` or aligned text report) to disk and/or stdout, along with basic run diagnostics (rows read, valid/skipped rows, elapsed time, and output path).

## 3. Edge Cases & Pitfalls to Guard Against

When generating a stat script, explicitly address these common failure modes:

* **Non-comma delimiters & quoted fields:** CSV/delimited exports often use `;`, `\t`, or `|` instead of `,`, and quoted fields may contain delimiters or escaped quotes (`""`). Avoid naive `.split(delim)` when quotes can wrap delimiters.
* **Dynamic column positions, blank lines & short/malformed rows:** Column order may vary across exports, blank lines or trailing separators may appear, and individual rows may have missing columns or non-numeric values. Match headers dynamically and skip/log bad rows instead of aborting the run.
* **Empty groups & percentile edge cases:** Some categories may have `0` or `1` record, or small sample sizes where index rounding could go out of bounds. Handle empty buckets cleanly and guard boundary indices in percentile calculations.
* **Unicode code points & script classification:** Counting characters or classifying text scripts (such as Han/Chinese vs. Latin/English vs. mixed) should account for supplementary Unicode code points and decide how spaces, punctuation, or symbols are treated rather than assuming 1 char = 1 code point or ASCII-only text.
* **Memory & I/O overhead on large datasets:** Storing full raw rows or unbuffered objects for hundreds of thousands or millions of rows can waste memory or time. Prefer streaming reads, buffered I/O, and storing only compact numeric metrics per group.
* **Ambiguous percentile definitions:** Different tools/languages can interpolate percentiles slightly differently. State the percentile convention used (e.g., linear interpolation) when precision matters.

## 4. Output Conventions

* **Keep outputs compact and easy to consume:** Output aggregated summary rows (group, metric, count, min, percentiles, max) rather than dumping millions of raw rows back out, saving a `CSV` or report file when repeated inspection or AI analysis is desired.
* **Keep derived files separate and non-destructive:** Never overwrite the raw input file; write summary outputs to a dedicated output path/folder and keep derived artifacts out of Git.
* **Surface data quality alongside metrics:** Include record counts per group and note skipped/malformed counts so tail values (`PR95`, `PR99`, `MAX`) are interpreted with proper sample context.
* **Reconcile the books and state the population:** Rows read must equal the sum of rows per group plus skipped rows (by reason), and group shares must add up to 100%. Groups must be mutually exclusive and collectively exhaustive, so use an explicit catch-all bucket rather than dropping unclassifiable rows. State whether the percentiles describe the whole file, a filtered subset, or a sample.

## 5. Generation Checklist (Before Finalizing the Script)

* \[ \] Does the script resolve target columns dynamically and handle delimiters/quotes/malformed rows safely?
* \[ \] Are empty groups, small samples, and Unicode/classification rules handled without index or encoding errors?
* \[ \] Is memory/I/O kept reasonable for the expected dataset size?
* \[ \] Are the computed statistics and output file location clearly presented?
* \[ \] Are any classification or percentile assumptions stated briefly for the user?