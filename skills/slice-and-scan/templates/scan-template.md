# Scan Blueprint (`scan-template.md`)

A language-agnostic structural blueprint for generating **Scan** scripts—scripts that batch-scan source files, workflow/config definitions, or structured records to locate target patterns or risks, trace cross-file/cross-step relationships, and emit compact reports or tables for inspection and reasoning.

Use this blueprint when generating a script for a **Scan** request. Adapt syntax, file traversal, matching rules, and output formatting to the user's requested language, runtime, and target artifacts.

## 1. When to Use This Blueprint

Use a **Scan** script when you want to find, classify, or connect information across many files or nested definitions without dumping raw files into context, especially when:

* **Locating patterns, APIs, or risk sites:** Finding specific statements, calls, annotations, or usage styles across a codebase and recording where each match occurs (such as file, symbol/scope, and line number).
* **Tracing cross-file or distant relationships:** Joining information split across multiple files or definitions—such as linking entity associations and join columns in one file to schema/table metadata in another file, or walking recursive workflow/job references (`.kjb`, `.ktr`, XML, configs) to spot missing files or cycles.
* **Classifying mixed conventions or access styles:** Categorizing occurrences across multiple patterns or tiers (such as different data-access approaches or framework usages) to understand what exists in a legacy module before changing or refactoring it.

## 2. Core Pipeline Structure

A well-formed scan script follows five stages, keeping scanning read-only, deterministic, and easy to inspect:

1. **CLI, Scope & Output Setup:** Accept target file(s) or root directory, optional filter/include/exclude patterns, and output path (defaulting to a dedicated derived-output folder or clear report file). Fail fast on missing/invalid required paths.
2. **Discovery & Content Preparation:** Traverse target files cleanly (filtering extensions and skipping irrelevant/generated/binary paths when appropriate), handle text encoding safely, and prepare text or parsed structures so comments, string literals, or formatting noise do not skew matches.
3. **Indexing & Cross-Reference Resolution (When Needed):** When matching requires looking up targets across files or steps, build an in-memory index (e.g., symbol/name/path -> metadata or normalized content), handle generics/wrappers or path variables/relative paths when applicable, and guard against visited/cycle loops.
4. **Pattern Matching, Classification & Record Emission:** Extract relevant fields or findings—putting both sides of a relationship on the same row/record when possible—and keep context (such as source/target names, scopes/classes/methods, lines, types, or snippets) without mutating source files.
5. **Report Writing & Summary Output:** Write structured output (`CSV` for compact tabular/relational analysis or clean text/table reports for human/AI reading) and print a concise summary (scanned file count, match/edge count, notable warnings/missing targets, and output path).

## 3. Edge Cases & Pitfalls to Guard Against

When generating a scan script, explicitly address these common failure modes:

* **Comments and string literals masking or faking matches:** Commented-out annotations/calls can produce false positives, while matching words inside log strings or SQL literals can cause false negatives or noise. Strip or separate comments/strings carefully when scanning source code.
* **Naive regexes on multi-line constructs or nested parentheses:** Simple single-line patterns or `([^)]+)` often break across newlines or nested parentheses/braces (such as multi-line annotations with nested attributes or complex method/class signatures). Match deliberately and avoid over-claiming exactness when heuristic parsing is used.
* **Generics, wrapper types & path/variable indirection:** Target types may appear inside wrappers (such as `List<Item>`, `Optional<Item>`, or framework containers), and file references may use environment/workflow variables, relative paths, or OS path separators (`\` vs. `/`). Normalize or resolve them before lookup.
* **Name collisions, duplicate symbols & missing targets:** Different packages or modules may define types or files with the same name, or referenced targets may be absent, unimported, or outside the scanned tree. Avoid silent crashes on missing lookups; record empty/unresolved status or warnings clearly.
* **Implicit defaults & silent false negatives:** Frameworks or configs often rely on defaults (such as default table names/schemas when omitted or inheritance/XML overrides outside simple rules). Document heuristic assumptions so findings are treated as a fast map rather than an infallible proof.
* **Circular references & encoding/read errors:** Recursive scans across workflow files or nested structures can loop infinitely without a `visited` set, and unreadable/non-UTF-8 files can abort a scan if unhandled. Track visited paths and skip/log unreadable files safely.

## 4. Output Conventions

* **Put related ends on the same record:** For cross-file or relationship scans, emit joined fields together on one row (e.g., source, relation/attribute, target, and target metadata) so downstream inspection or AI reasoning does not have to stitch distant files back together.
* **Keep outputs derived and separate:** Write reports or CSVs to a dedicated output directory or explicit output path rather than modifying scanned source files, and avoid committing derived outputs to Git.
* **Choose format to fit downstream use:** Prefer `CSV` with clear headers when feeding structured rows/relationships into further analysis or AI prompts, and structured text tables/summaries when human readability and counts/snippets are primary.

## 5. Generation Checklist (Before Finalizing the Script)

* \[ \] Is the scan strictly read-only with respect to the target codebase/files?
* \[ \] Are comments, strings, multi-line constructs, generics/paths, and missing/cyclic references handled appropriately for the target format?
* \[ \] Are cross-file/related details surfaced together clearly rather than left scattered?
* \[ \] Does the script write a saved report/table and print a clear summary of counts and output location?
* \[ \] Are heuristic shortcuts or potential silent misses stated briefly so the user knows the scan's limits?