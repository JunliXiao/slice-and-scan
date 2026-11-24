# Multi-Module Reference (`multi-module.md`)

How to apply `slice-and-scan` across multi-module systems or multiple repositories without building a fragile, bloated cross-repo script.

## 1. Default Habit: Single-Module-First

Start with **one module, repository, or dataset at a time**:

* **Keep each script focused:** A script tailored to one module's directory layout, language, conventions, and file types stays small, fast to generate, and easy to verify.
* **Preserve context clarity:** Scoped inputs and outputs keep both human inspection and AI reasoning sharp without mixing unrelated conventions or path assumptions into a single run.

## 2. Why One Monolithic Cross-Repo Script Usually Backfires

When a change spans multiple repositories or mixed stacks (for example, SQL/ETL scripts or workflow files in one repo, batch/service code in another, domain models in a third, and backend + UI code in a fourth), cramming everything into one script tends to create friction:

* **Heterogeneous syntax and structure:** Different repos often use different languages, frameworks, file extensions, comment styles, and naming conventions, turning a simple script into a tangled multi-branch parser that is harder to debug.
* **Different roots, paths, and runtimes:** Separate repositories have their own root paths, encodings, build/run assumptions, and output targets.
* **Coupled maintenance:** If one repo's layout changes, a combined script can break across unrelated modules.

## 3. Recommended Workflow for Cross-Module Changes

When assessing impact or checking field/contract alignment across multiple modules:

1. **Scope the shared contract or target first:** Write down the target field(s), entity/table names, event/message keys, or contract touchpoints and any known module context.
2. **Generate and run one small script per module/layer:** Have each script inspect its own module/repo (or sub-layer, such as backend vs. frontend) and emit a compact, structured summary (`CSV` or clear table/list) into that module's ignored output folder, including the scanned, matched, skipped, and unresolved counts from Principle 6, so a missing touchpoint is never confused with an unscanned one.
3. **Correlate the compact summaries:** Feed the resulting per-module summaries—rather than raw source trees from every repo—into analysis to check field naming, mapping, transformations, or missing touchpoints across boundaries.

## 4. When a Multi-Part Check Is Worth Keeping

Only keep a multi-script check (such as per-module scan scripts plus a lightweight correlation step) when:

* That cross-module change pattern recurs frequently in your workflow (for example, repeated cross-system field additions or contract audits).
* Each module's conventions are already well understood and reasonably stable.
* The parts stay modular—each module has its own small scanner/output, and any correlation step simply joins or compares their compact outputs rather than merging all parsing logic into one file.