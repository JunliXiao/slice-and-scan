# Example: Cross-File Association Scan (`association-scan.md`)

A concrete problem-to-script walkthrough showing how a **Scan** script resolves relationships spread across multiple source files and writes both ends onto a single compact row for easier inspection and reasoning.

## Problem Context

* **Scenario:** A legacy Java codebase uses persistence annotations (such as `@Table` and `@JoinColumn`) across separate entity classes. Reading every entity file individually wastes context and makes it easy to lose track of which join column or association points to which target schema and table.
* **Mode:** `scan` (cross-file relationship extraction into a compact table).
* **Target / Runtime:** A single-file standalone script (for example, Java or Python) run against a source directory, writing a derived `CSV` report to a dedicated output folder.

## Key Design Moves

1. **Strip comments before indexing or matching:** Remove line (`//`) and block (`/* ... */`) comments up front so commented-out annotations or old mappings do not create false positives.
2. **Build an in-memory symbol index:** Walk target source files once and index cleaned content by symbol/class name so association lookups can resolve across files in memory rather than re-scanning disk per reference.
3. **Unwrap common container/generic types:** When a field type is wrapped (such as `List<Child>` or `Optional<Child>`), extract the inner target type name before looking up its table metadata.
4. **Join source and target metadata onto one row:** Emit fields such as package/source symbol, source schema/table, association type, join attribute/column, target symbol, and resolved target schema/table together on each row so downstream reasoning sees the full link at a glance.

## Sample Output

Entity mapping (one row per class):

```csv
Package,Class,TableSchema,TableName
com.example.hr.model,Employee,HR,EMPLOYEES
com.example.hr.model,Department,HR,DEPARTMENTS
com.example.hr.model,Position,HR,POSITIONS
com.example.hr.model.payroll,PayrollRecord,PAY,PAYROLL_RECORDS
com.example.hr.model,EmployeeExtra,,HR.EMPLOYEE_EXTRA
```

Association report (both ends of each link on one row):

```csv
Package,Class,TableSchema,TableName,AssociationType,JoinColumn,AssociatedClass,JoinTableSchema,JoinTableName
com.example.hr.model,Employee,HR,EMPLOYEES,ManyToOne,DEPARTMENT_ID,Department,HR,DEPARTMENTS
com.example.hr.model,Employee,HR,EMPLOYEES,ManyToOne,POSITION_ID,Position,HR,POSITIONS
com.example.hr.model,Department,HR,DEPARTMENTS,OneToMany,DEPARTMENT_ID,Employee,HR,EMPLOYEES
com.example.hr.model.payroll,PayrollRecord,PAY,PAYROLL_RECORDS,ManyToOne,EMPLOYEE_ID,Employee,HR,EMPLOYEES
com.example.hr.model,Employee,HR,EMPLOYEES,OneToMany,EMPLOYEE_ID,LeaveRequest,(unresolved),(unresolved)
```

Two rows deserve attention: `EmployeeExtra` has an empty `TableSchema` because the schema was written inside the table name, and the `LeaveRequest` link is marked `(unresolved)` because the target class was not found in the scanned tree. Unresolved rows are kept and counted, never dropped.

## Edge Cases & Limitations

* **Name collisions across packages:** Indexing solely by unqualified class name can hit the wrong class if two packages define the same name. Scope lookups with imports/packages or note potential ambiguities.
* **Implicit defaults and non-annotation overrides:** Omitted schema/table names may follow framework naming defaults, while XML mappings, inheritance (`@MappedSuperclass`), or custom naming strategies may not appear in simple annotation regexes. Treat the `CSV` as a fast structural map, not a complete proof.
* **Schema embedded in the table attribute:** An annotation such as `@Table(name = "HR.EMPLOYEE_EXTRA")` carries the schema inside the table name. A scan that only reads a separate `schema` attribute leaves `TableSchema` empty and puts `HR.EMPLOYEE_EXTRA` in `TableName`. Split on the dot or flag such rows, so an empty schema cell is never mistaken for "default schema" or "no schema".

## When to Adapt This Pattern

* Mapping entity-to-table or foreign-key/join links across model files.
* Tracing cross-file references, event publishers/listeners, or service-to-client/endpoint links where the two sides live in separate files.