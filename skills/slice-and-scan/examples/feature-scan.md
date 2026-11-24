# Example: Comment-Safe Feature & Convention Scan (`feature-scan.md`)

A concrete problem-to-script walkthrough showing how a **Scan** script locates specific usages or classifies mixed conventions across a codebase while preserving enclosing scope context and avoiding comment/literal noise.

## Problem Context

* **Scenario:** A legacy Java module has accumulated multiple data-access styles or raw SQL/statement usages over time. Before refactoring or tightening standards, you want to find where specific patterns appear, classify usage categories by priority, and record the enclosing class, method, and line number.
* **Mode:** `scan` (pattern location and convention classification into a structured report).
* **Target / Runtime:** A standalone non-interactive script (such as single-file Java or Python) scanning a target directory and writing a report file without touching source files.

## Key Design Moves

1. **Filter block and line comments line-by-line:** Track multi-line block comment state (`/* ... */`) across lines and strip `//` comments so commented-out code or remarks are not reported as active usages.
2. **Sanitize string literals before pattern matching:** Neutralize quoted string contents (such as `"..."`) before running usage/keyword checks so words appearing inside log messages or SQL text strings do not cause false hits or misclassifications.
3. **Track enclosing class and method context:** Maintain current class and method names as lines are scanned (filtering out control-flow keywords like `if`, `for`, `while`, `switch`, `catch` that look like method calls) so every finding carries actionable location context.
4. **Apply explicit priority rules for overlapping patterns:** When broader/higher-level patterns and lower-level primitives can both match a line (for example, framework/ORM/template APIs vs. raw JDBC/connection calls), evaluate in a deliberate priority order and count/label each hit cleanly.
5. **Emit both per-hit rows and summary counts:** Record class, method, line number, category/type, and code snippet in a saved report file, paired with a summary count by category and a console status line.

## Sample Output

```text
===============================================================================================
 Database Access Technique Summary
 Scan target: C:\workspace\hr_portal\src\com\example\hr
===============================================================================================
Class Name       | Method Name            | Line No | Access Type         | Code Snippet
-----------------------------------------------------------------------------------------------
UnknownClass     | OutsideMethod          | 5       | Hibernate / JPA     | import javax.persistence.EntityManager;
EmployeeDAO      | findEmployeeById       | 42      | Hibernate / JPA     | EntityManager em = this.getHrEntityManager();
EmployeeDAO      | findEmployeeById       | 44      | Hibernate / JPA     | return em.find(Employee.class, employeeId.longValue());
PayrollDAO       | sumSalaryByDepartment  | 88      | Spring JdbcTemplate | return jdbcTemplate.queryForObject(sql, Long.class, deptId);
LegacyReportDAO  | exportHeadcount        | 131     | Raw JDBC            | Statement stmt = conn.createStatement();
LegacyReportDAO  | exportHeadcount        | 133     | Raw JDBC            | ResultSet rs = stmt.executeQuery(sql);
...
-----------------------------------------------------------------------------------------------
Files scanned: 128 | Files skipped (unreadable): 0 | Files with hits: 17 | Total hits: 214
  Hibernate / JPA      171
  Spring JdbcTemplate   29
  Raw JDBC              14
```

The sample is an excerpt; the real report lists every hit. Hits found outside any method (such as `import` lines) are labeled `UnknownClass` / `OutsideMethod` rather than dropped, so the line number still leads back to the source. Because Spring `JdbcTemplate` is checked before raw JDBC primitives, a line is counted under exactly one category.

## Edge Cases & Limitations

* **Multi-line statements, inner classes, and overloads:** Tracking class/method names via lightweight line patterns is a heuristic—anonymous classes, nested/inner classes, multi-line signatures, or unusual formatting can occasionally leave scope approximate.
* **Escaped quotes and multi-line strings:** Simple quote stripping can be tripped by escaped quotes (`\"`) or multi-line string blocks if not handled carefully.

## When to Adapt This Pattern

* Auditing raw SQL/statement calls, deprecated APIs, or security/coding-convention hotspots across a module.
* Inventorying mixed persistence/access styles or framework conventions before a migration or refactor.