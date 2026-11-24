# Slice Blueprint (`slice-template.md`)

A language-agnostic structural blueprint for generating **Slice** scripts—scripts that split large monolithic files into smaller units, extract scoped subsets, or normalize/derive fields into intermediate outputs for inspection or downstream `scan` / `stat` steps.

Use this blueprint when generating a script for a **Slice** request. Adapt syntax, I/O APIs, and boundary rules to the user's requested language, runtime, and input format.

## 1. When to Use This Blueprint

Use a **Slice** script when the raw input is too large, too coarse, or too irregular to inspect or analyze directly, and you need to:

* **Split large files by structural markers:** Break monolithic source, SQL, LOB, dump, or export files into per-unit files (such as per `PROCEDURE`, `FUNCTION`, `PACKAGE`, or `PACKAGE BODY`).  
* **Extract a scoped subset:** Filter lines, records, blocks, or paths matching specific criteria into a smaller working file.  
* **Normalize or derive views/columns:** Split delimited/compound values, normalize casing/whitespace, or produce parallel derived representations (such as tokenized vs. full-string views along with length/count metrics) for downstream inspection or `stat` analysis.

## 2. Core Pipeline Structure

A well-formed slice script follows five stages, prioritizing streaming/low-memory execution when files may be large:

1. **CLI & Config Parsing:** Accept input path(s), optional output directory, optional run/row limit for smoke testing, and format/mode options. Fail fast with a clear message if required inputs are missing or invalid.  
2. **Boundary & Format Detection:** Identify delimiters, encoding, line endings, header rows, or structural start/end markers before processing records or lines.  
3. **Stateful or Record-Level Slicing:**  
   * *Block/stream splitting:* Track state across lines/chunks (e.g., inside vs. outside a routine/package block, handling preamble text before the first marker and flush/close behavior at EOF or next marker).  
   * *Row/field slicing & derivation:* Parse rows cleanly (respecting quotes/delimiters rather than naive splitting when needed), extract target fields, and build scoped or derived outputs.  
4. **Safe Output Writing:** Create output directories safely, avoid overwriting unrelated files unintentionally, handle empty/malformed/short rows gracefully without crashing the whole run, and keep derived artifacts separate from source files.  
5. **Summary Reporting:** Print concise progress/completion counts (files/rows read, slices/records emitted, skipped/malformed counts, elapsed time, and output path(s)) to stdout/stderr.

## 3. Edge Cases & Pitfalls to Guard Against

When generating a slice script, explicitly address these common failure modes:

* **Case sensitivity & spacing variations:** Keywords or headers may vary in case (`PROCEDURE` vs. `procedure`, `NAME` vs. `name`), extra whitespace, or line endings (`\r\n` vs. `\n`). Normalize case/whitespace when matching without mutating original content unless intended.  
* **Preamble, mid-line markers & trailing fragments:** A marker (such as `CREATE OR REPLACE PACKAGE` or a new block header) may appear mid-line or after leading text, or there may be content before the first marker and unclosed content at EOF. Decide explicitly whether to prepend, discard, or flush remaining buffers.  
* **Overly greedy end markers:** A naive end check (such as matching `END;` inside nested `IF ... END IF;` or `CASE ... END;`) can split blocks prematurely. Match scoped closing markers when possible, or document the heuristic limitation clearly.  
* **Quotes, delimiters & full-width/variant separators:** Delimited text and identifiers often contain quoted separators, escaped quotes (`""`), or variant punctuation/whitespace (such as full-width spaces, ideographic delimiters, or mixed separators). Avoid naive `.split(",")` when quoted fields or custom delimiters are present.  
* **Missing columns, empty lines & malformed records:** Skip blank lines cleanly and guard index accesses so one short or malformed row does not throw an out-of-bounds or parse exception that aborts a multi-hundred-thousand-row run.  
* **Path & filename collisions:** Sanitize generated names, handle missing parent directories or path separators consistently across OSes, and avoid collisions when multiple items share a name or differ only by case/type (e.g., spec vs. body).

## 4. Output Conventions

* **Keep outputs derived and inspectable:** Write split files or derived tables (`CSV`, `.sql`, `.txt`, etc.) to a dedicated output folder (such as `.slice-scan-out/` or a user-specified target directory) rather than mutating source files in place.  
* **Preserve traceability:** When splitting or deriving records, retain enough context in filenames, headers, or columns (such as source name, type marker, sequence/line info, or derived format/length fields) to trace results back to the source.  
* **Support dry/limited runs:** When processing large files or exports, support an optional limit or clear progress logging so the user can verify a small slice before running over the full dataset.

## 5. Generation Checklist (Before Finalizing the Script)

* \[ \] Does the script avoid unnecessary external libraries and fit the user's stated language/runtime?  
* \[ \] Are input paths, output paths, and missing/malformed records handled safely?  
* \[ \] Are boundary/delimiter/case edge cases explicitly handled rather than assumed away?  
* \[ \] Does the script output a clear summary of what was sliced and where the outputs were written?  
* \[ \] Are any remaining assumptions or heuristic limitations stated briefly for the user?