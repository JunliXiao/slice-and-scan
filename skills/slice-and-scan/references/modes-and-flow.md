# Modes and Flow (`modes-and-flow.md`)

How `slice`, `scan`, and `stat` work on their own or chained together, and which mode answers which kind of question.

## 1. The Three Modes at a Glance

| Mode | Primary Question Answered | Typical Input | Typical Output |
| ----- | ----- | ----- | ----- |
| **Slice** | How can I break this down or reshape it into cleaner pieces/views? | Monolithic files (SQL, LOB, DEL, logs) or bulky/compound records | Split files, scoped subsets, or normalized/derived rows/columns |
| **Scan** | Where do targets/patterns sit, or how do pieces connect across files/steps? | Source trees, XML/workflow files, configs, or structured records | Match lists with scope/line context, classification summaries, or joined cross-file tables |
| **Stat** | What are the typical values, ranges, and tail shapes across groups? | Tabular/delimited data or outputs from a `slice` / `scan` step | Grouped counts, min/max bounds, and percentiles (`PR25`, `PR50`, `PR75`, `PR95`, `PR99`) |

## 2. Standalone Usage (Most Common)

Most tasks only need **one** mode:

* **Use `scan` alone** when you want to map relationships across files (such as entity annotations and target tables), trace recursive workflow references (such as `.kjb` / `.ktr` chains), or inventory patterns/conventions (such as raw statements or data-access styles) before changing code.  
* **Use `slice` alone** when a single file is too big or awkward to read or diff as one blob—for example, splitting a giant PL/SQL or package export into individual routine or package files so you can inspect the ones that matter.  
* **Use `stat` alone** when you already have a table or delimited dataset and want empirical distribution numbers—such as string lengths or token counts grouped by script/category—to check invariants or size buffers/limits.

## 3. Chained Usage (Composable Steps)

When a single step does not expose what you need cleanly, chain modes loosely rather than forcing a rigid pipeline:

* **`slice` \-\> `scan`:** Split a huge monolithic file or dump into smaller scoped files or subsets first so scanning is faster, cleaner, and easier to inspect per unit.  
* **`slice` \-\> `stat`:** Normalize or derive fields/views first (for example, splitting compound text into token vs. full-string views or computing helper counts), then run `stat` to inspect distributions and tail percentiles across groups.  
* **`scan` \-\> `stat`:** Scan a codebase or dataset to emit a structured table of findings, classifications, or extracted metrics, then run `stat` on a numeric or grouped column to see concentration and heavy tails.

You do not have to run all three steps every time. Stop as soon as the intermediate or final output answers your question.