# Example: Monolithic File & Field Slicing (`file-slice.md`)

A concrete problem-to-script walkthrough showing how a **Slice** script breaks large monolithic files into smaller scoped units or derives normalized views/columns so downstream inspection or `stat` analysis stays fast and manageable.

## Problem Context

* **Scenario:** A large SQL/PLSQL/LOB/DEL export file bundles many procedures, functions, or package specs/bodies into one huge file, or a large delimited dataset has compound/raw string values that need normalized token vs. full-string views before measuring distributions.  
* **Mode:** `slice` (splitting large files into scoped units or normalizing/deriving intermediate views).  
* **Target / Runtime:** A standalone, non-interactive script (such as Java, Python, or PowerShell) reading from an input path and writing split files or derived tables into a dedicated output directory.

## Key Design Moves

1. **Track structural markers and state transitions:** When splitting routines or packages (such as `PROCEDURE`, `FUNCTION`, `CREATE OR REPLACE PACKAGE`, or `CREATE OR REPLACE PACKAGE BODY`), track the active state and write each unit to its own file. Write a package spec and its body as separate files with the type marked in the filename, since they share the same name.
2. **Derive parallel views cleanly for downstream analysis:** When normalizing fields (such as splitting/tokenizing delimited text vs. keeping the full normalized string), emit clear derived columns/files and carry forward useful metrics (such as token count or string length) without mutating the source file.  

## Sample Output

Input: one export file with all packages concatenated. Note that `END HR.LEAVE_SERVICE` has no semicolon or trailing newline, so the next `CREATE` marker starts mid-line:

```text
CREATE OR REPLACE PACKAGE HR.LEAVE_SERVICE AS
	PROCEDURE approveLeave(v_returnCode OUT INT, v_requestId INT);
END HR.LEAVE_SERVICECREATE OR REPLACE PACKAGE BODY HR.LEAVE_SERVICE AS
	PROCEDURE approveLeave(v_returnCode OUT INT, v_requestId INT)
	AS
	BEGIN
		UPDATE HR.LEAVE_REQUEST SET STATUS = 'APPROVED' WHERE REQUEST_ID = v_requestId;
		v_returnCode := 0;
	END approveLeave;
END HR.LEAVE_SERVICECREATE OR REPLACE PACKAGE HR.ATTENDANCE_LOG
AS
	...
```

Output: one spec file and one body file per package, with the type marked in the filename:

```text
.slice-scan-out/
|-- HR.LEAVE_SERVICE_spec.sql
|-- HR.LEAVE_SERVICE_body.sql
|-- HR.ATTENDANCE_LOG_spec.sql
|-- HR.ATTENDANCE_LOG_body.sql
|-- HR.PAYROLL_SERVICE_spec.sql
`-- HR.PAYROLL_SERVICE_body.sql
```

Run summary printed by the script:

```text
Lines read: 2380 | Packages found: 3 | Spec files: 3 | Body files: 3
Unpaired (spec without body, or the reverse): 0 | Preamble lines discarded: 0
Output: .slice-scan-out/
```

Each slice file starts at its `CREATE` marker and ends just before the next one, so every file can be opened and read on its own. An unpaired spec or body is reported, never silently skipped.

## Edge Cases & Limitations

* **Start marker glued to the previous end marker:** A package's final `END <name>` may have no semicolon or trailing newline, so the next `CREATE OR REPLACE PACKAGE` begins mid-line. Match the marker anywhere in the line, not only at the start.

## When to Adapt This Pattern

* Splitting large PL/SQL, SQL, or dump/LOB files into per-object or per-routine files before reading or diffing.  
* Normalizing or deriving text/field representations (`tokened` vs. `full-string`, filtered subsets) prior to running a `stat` script.