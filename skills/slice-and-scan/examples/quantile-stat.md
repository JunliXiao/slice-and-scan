# Example: Grouped Quantile & Distribution Analysis (`quantile-stat.md`)

A concrete problem-to-script walkthrough showing how a **Stat** script groups records and computes summary counts, min/max bounds, and percentiles (`PR25`, `PR50`, `PR75`, `PR95`, `PR99`) to understand data shapes, spot heavy tails, and guide capacity or performance decisions.

## Problem Context

* **Scenario:** A large delimited dataset contains text/name fields and numeric length or token/segment metrics. Reading raw rows cannot show typical vs. extreme sizes across character/script groups, so you need grouped distribution percentiles to confirm invariants or size buffers/limits.  
* **Mode:** `stat` (grouped aggregation and percentile distribution summary).  
* **Target / Runtime:** A standalone non-interactive script (such as Java or Python) reading one or more `CSV`/delimited files and printing or saving a compact distribution summary.

## Key Design Moves

1. **Classify records into clear buckets:** Group items into meaningful categories—such as character/script groups (e.g., Han_only vs. Non_Han vs. Mixed_or_other) while ignoring expected whitespace or basic punctuation as defined for the task—and store only compact numeric values per group.  
2. **Sort per bucket and compute ordered summary metrics:** Sort each group's numeric list and compute `COUNT`, `MIN`, `MAX`, and percentiles (`PR25`, `PR50`, `PR75`, `PR95`, `PR99`) using a consistent interpolation method.  
3. **Emit a compact summary table:** Output aggregated rows (group, metric, count, min, percentiles, max) to a saved report and/or console summary so tail behavior is immediately visible.

## Sample Output

```csv
CATEGORY,METRIC,COUNT,MIN,PR25,PR50,PR75,PR95,PR99,MAX
Han_only,STRING_LENGTH,41250,1.0,3.0,3.0,3.0,4.0,6.0,12.0
Non_Han,STRING_LENGTH,12480,3.0,8.0,12.0,15.0,21.0,25.0,34.0
Mixed_or_other,STRING_LENGTH,9310,0.0,6.0,11.0,15.0,22.0,27.0,38.0
```

```text
Rows read: 63100 = 41250 + 12480 + 9310 (classified) + 60 (skipped: blank 10, short 20, other 30)
Shares of classified rows: Han_only 65.4% | Non_Han 19.8% | Mixed_or_other 14.8% (total 100.0%)
Population: the full input file, no filtering or sampling
```

The summary is three rows, while the input it was computed from can hold tens of thousands of records. Read the `COUNT` column alongside the percentiles: a heavy tail like `PR99` or `MAX` only means something when the group has enough records.

## Edge Cases & Limitations

* **The mixed bucket is a catch-all:** Besides names that genuinely mix Han and other characters, rows with nothing classifiable (empty strings, or only whitespace and basic punctuation) also land in the mixed bucket, which can pull its `MIN` down to 0. Digits, symbols, and other scripts count as non-Han, so the "pure" group is really "no Han characters". State these rules explicitly, since they change every percentile.

## When to Adapt This Pattern

* Measuring string lengths, token/segment counts, payload sizes, or latency/batch metrics across categories before tuning limits or queries.  
* Checking data invariants and tail distributions (`PR95`, `PR99`, `MAX`) on raw exports or outputs produced by a prior `slice` or `scan` step.