# SQL Command Categories, Query Processing, and Writing Your First Table

This session turns from database theory to hands-on SQL: how SQL commands are classified, what happens behind the scenes when a query runs, the main Oracle data types, and a full walkthrough of creating a `Students` table and inserting real rows into it.

## Key Takeaways

- SQL isn't one flat list of commands — it's split into functional categories: **DDL** (define structure), **DML** (manipulate data), **DQL** (query/retrieve data), **DCL** (control access), and **TCL** (manage multi-step transactions) — and recognizing which category a task falls into is core to understanding SQL as a language.
- Every SQL statement you run passes through three stages inside the DBMS — **parsing** (syntax/existence check), **optimization** (choosing the fastest execution plan), and **execution** (actually running the plan and returning rows) — so the database is doing real decision-making before your query results ever appear.
- Choosing the right **data type and size** for each column (fixed vs. variable-length text, numeric precision and scale, dates, large objects) directly affects storage efficiency and data integrity, and Oracle enforces **constraints** like `NOT NULL` and `DEFAULT` values right at table-creation time.

## 1. What Is SQL, in Practical Terms?

**SQL (Structured Query Language)** is the language used to talk to the *data* stored inside a database — not to the DBMS software itself, but to the data it manages. The DBMS receives SQL statements, processes them, and carries out whatever the statement asked for against the underlying tables. Since the relational model organizes everything into tables built from columns (attributes) and rows (records), essentially all of SQL's job is either shaping those tables, filling them with data, retrieving data back out, or controlling who's allowed to do any of that.

Practically, everything you'd ever want to do to a database — create it, build a new table inside it, insert rows, update or delete existing rows, query for specific data, grant or revoke someone's access, or manage a multi-step transaction — is expressed as SQL.

## 2. Technical Flow: How SQL Commands Are Classified, and How One Gets Executed

### The Five Categories of SQL Commands

1. **DDL — Data Definition Language.** Commands used to define or modify the *structure* of database objects — creating a table (specifying its name, columns, and constraints), or later altering that structure (adding/removing a column, adding a constraint).
2. **DML — Data Manipulation Language.** Once a table's structure exists, DML commands manipulate the actual data inside it: inserting new rows, updating existing ones, or deleting rows.
3. **DQL — Data Query Language.** Commands used specifically to retrieve or query existing data — the lecture frames this as its own category focused purely on fetching, e.g., pulling back all customers registered after a certain date, or all students who passed a given course.
4. **DCL — Data Control Language.** Commands that manage user access and permissions — granting a specific privilege to a specific user, or revoking a privilege that was previously granted. Even the act of assigning or removing permissions happens through SQL statements.
5. **TCL — Transaction Control Language.** Commands that manage a **transaction**: a sequence of multiple steps that together accomplish one task (e.g., a bank transfer, which requires checking a balance, debiting one account, crediting another, and logging the result). TCL commands let you mark where a transaction begins and ends, and decide whether to make its changes permanent (**commit**) or undo them if something went wrong (**rollback**).

### How a Single SQL Statement Actually Runs

1. **Parsing.** The DBMS first checks the statement the way a compiler checks code: is it written with valid, complete syntax, in the correct clause order, and do the tables/columns it references actually exist in the database? A malformed statement or a reference to a nonexistent table fails right here.
2. **Optimization.** A single request can often be satisfied by more than one internal execution strategy — much like there's more than one way to travel to a destination (by car, by plane, with a group). The **optimizer** evaluates the available options and picks the plan expected to be fastest, weighing factors like the expected data volume involved and where the data physically lives (a table might be mirrored in more than one physical location, and the optimizer accounts for that too).
3. **Execution.** The database carries out the chosen plan. For a query that joins multiple tables, this can mean executing several intermediate steps in sequence — each returning an intermediate result set that feeds into the next step — until a final result set is produced and returned to the user.

## 3. Why Do We Need Structured SQL Categories and a Query-Processing Pipeline?

- **Clarity and mental organization** — grouping commands by purpose (defining structure vs. manipulating data vs. querying vs. controlling access vs. managing transactions) makes SQL far easier to learn and reason about than treating it as one undifferentiated list of keywords.
- **Safe, granular access control** — because permission management (DCL) is its own category, a database administrator can grant or revoke very specific privileges to specific users, rather than giving blanket access to everyone who can log in.
- **Data integrity through validation before execution** — the parsing stage catches malformed or invalid statements before anything is touched, preventing errors from cascading into the actual data.
- **Performance at scale** — the optimization stage matters precisely because data can be large and can live in more than one physical location (including mirrored copies); picking a smart execution plan is what keeps queries fast as data grows.
- **Reliable multi-step operations** — real-world tasks (like transferring money) are rarely a single action; TCL exists so a group of related steps either all succeed together or can be safely rolled back if something goes wrong partway through.

## 4. Key Comparisons: SQL Command Categories vs. Treating SQL as "Just One Language"

### Why can't we just treat every SQL statement the same way?

1. **Different categories change what's actually allowed.** A `SELECT` (DQL) only reads data and can't alter anything, while an `INSERT`/`UPDATE`/`DELETE` (DML) actively changes stored data — conflating them risks running a data-altering command when only a read was intended.
2. **Structure changes and data changes have very different blast radii.** A DDL command (like adding or dropping a column) reshapes the table itself and can affect every row and every application using it, while a DML command typically affects only the specific rows it targets — treating them the same way ignores how much more disruptive a structural change can be.
3. **Access control needs to be separable from data operations.** If permission-granting weren't its own category (DCL), it would be harder to reason about and audit who can do what — keeping it distinct is what allows fine-grained, revocable privileges.
4. **Multi-step business processes need explicit boundaries.** Without TCL's concept of a transaction (with a clear begin/commit/rollback), a multi-step task like a bank transfer could be left half-completed if something fails midway — a single, ungrouped sequence of DML statements offers no way to guarantee all-or-nothing execution.

*(Accuracy note: some references also list a sixth category, DRL/"Data Retrieval Language," for `SELECT`, or fold `SELECT` into DML instead of a separate DQL category — naming conventions vary slightly across textbooks and vendors, but the functional distinctions the lecture draws — structure vs. data vs. retrieval vs. access vs. transactions — match how SQL is taught and used industry-wide today.)*

## 5. Deep Dive: Table Creation Rules, Oracle Data Types, and a Full Hands-On Example

### Rules for Naming Tables and Columns

- A table or column name **cannot start with a number or a special character** — it must begin with a letter (this mirrors the same restriction used for variable names in general-purpose programming).
- Numbers **are** allowed later in the name, just not as the first character.
- Names can be **1 to 30 characters** long. The lecture recommends keeping names short but *descriptive enough* that another developer can understand a table's purpose without needing to open it and investigate — a habit carried over from general software engineering practice.
- Allowed characters: uppercase and lowercase **letters (A–Z)**, **digits (0–9)**, plus the underscore (`_`), dollar sign (`$`), and hash/pound sign (`#`).
- A table name **cannot be duplicated within the same database**, though the same name can be reused across different databases (schemas).
- Table names are conventionally written in **plural form**, following the wider industry convention already introduced in the previous session.

### Anatomy of a `CREATE TABLE` Statement

The general shape demonstrated is:

```sql
CREATE TABLE table_name (
  column_name DATATYPE constraints,
  column_name DATATYPE constraints,
  ...
);
```

Key points the lecture emphasizes about this syntax:

- Unlike general-purpose programming (where you'd typically write the type before the variable name), SQL writes the **column name first, then its data type**.
- Columns are separated by commas, with **no trailing comma** after the last column.
- The whole block is a single **statement**, and it must end with a **semicolon (`;`)** to mark that the statement is complete.
- Optionally, a table can be created inside a specific **schema** by prefixing the table name with `schema_name.table_name` — necessary when accessing or creating tables outside your own default schema (for example, a colleague's schema), but unnecessary when working inside your own.
- Once written, the statement is executed by running it (the lecture uses the **Run** button in Oracle Live SQL, as introduced in the previous session).

### Oracle Data Types Covered

| Data Type | What It Stores | Key Details |
|---|---|---|
| **CHAR(size)** | Fixed-length text | Reserves the full specified size in memory regardless of actual input length (max 2,000 characters/bytes in this Oracle version) — wastes space if the real data is shorter than the declared size. |
| **VARCHAR2(size)** | Variable-length text | Only uses as much storage as the actual data needs, up to the declared max; the excess reserved space isn't wasted the way it is with `CHAR`. Oracle-specific; the lecture notes that plain `VARCHAR` (without the "2") is the portable, cross-vendor equivalent if code needs to run on non-Oracle systems. Max size in this context: **4,000 characters/bytes** (a much higher limit — 32,767 bytes — exists under Oracle's "extended" string-size mode, a detail beyond the scope of this intro lecture). |
| **NUMBER(precision, scale)** | Numeric values, with or without decimals | `precision` is the total number of digits (1 to 38); `scale` is how many of those digits appear after the decimal point (−84 to 127, per Oracle's official range, matching the lecture). If more decimal digits are entered than the scale allows, Oracle **rounds** the value to fit — but it will still reject a value whose *whole-number* part exceeds the specified precision. |
| **DATE** | Calendar dates (and, depending on entry, time) | The demo enters dates in a `DD-MON-YYYY`-style format. Related, more advanced types mentioned only briefly include ones for time zones and fractional seconds (down to milliseconds), useful in domains needing very high timestamp precision, like stock trading or sensor logging. |
| **LONG** | Large variable-length text | Can hold up to about 2 gigabytes of character data — used rarely, and generally considered a legacy type in modern Oracle practice, where `CLOB` is preferred for large text. |
| **CLOB** | Character Large Object | For large amounts of text, up to about 4 gigabytes. |
| **NCLOB** | National Character Large Object | Same purpose as `CLOB`, but for storing Unicode text in non-English scripts (e.g., Arabic, Urdu). |
| **BLOB** | Binary Large Object | For storing arbitrary binary data — images, video clips, audio — up to about 4 gigabytes. |
| **BFILE** | Binary File pointer | For referencing external binary files (e.g., design files) rather than storing the bytes directly inside the database. |
| **ROWID** | Internal row identifier | A pseudo-column-style value that can be used to represent a row's sequential position/identity within a table. |

*(Accuracy note: the size limits above reflect Oracle's traditional/"standard" configuration, which matches what the lecture describes; a newer "extended" string mode raises `VARCHAR2`'s limit considerably, but that's an advanced configuration detail outside this lecture's intro-level scope.)*

### Column Constraints Demonstrated

While defining the `Students` table, two constraint concepts are introduced directly through practical decisions:

- **`NOT NULL`** — marks a column as mandatory; the DBMS will reject any row insert that leaves that column empty. Applied to the student ID (a record must have one to make sense) and the student's name.
- **`DEFAULT`** — supplies an automatic value when the user doesn't provide one for that column. Two examples are used: the registration-date column defaults to **`SYSDATE`** (Oracle's built-in function returning the current date), and the gender/sex column defaults to a fixed value (`'M'`) appropriate for an all-male student population in this example, using single quotes around text/character literals in Oracle SQL (not the double quotes used in some other languages).

### Full Walkthrough: Building and Populating a `Students` Table

The hands-on example builds a table step by step, reasoning through each column:

```sql
CREATE TABLE Students (
  Student_ID NUMBER(9) NOT NULL,
  First_Name VARCHAR2(50) NOT NULL,
  GPA NUMBER(3,2),
  Registration_Date DATE DEFAULT SYSDATE,
  Gender CHAR(1) DEFAULT 'M'
);
```

Reasoning walked through live:

- **`Student_ID`**: numeric, matching the university's real 9-digit student ID format, and marked mandatory since a student record without an ID doesn't make sense.
- **`First_Name`**: text, capped at a reasonable length (50 characters in the demo), and also marked mandatory. The lecture notes in passing that splitting a full name into first/last (and, more generally, splitting composite data like addresses into city/street/etc.) pays off later, because it makes searching and filtering far easier — a lesson that carries forward to normalization practices covered afterward.
- **`GPA`**: numeric with 3 total digits and 2 after the decimal point (e.g., `3.75`), left optional. When more decimal digits are entered than the scale allows, Oracle rounds instead of rejecting the value — but an attempt to enter a value whose whole-number part is too large for the declared precision (e.g., entering `13.7` when only one digit is allowed before the decimal) is rejected outright.
- **`Registration_Date`**: a date, defaulting to `SYSDATE` (today's date) if the person entering data doesn't supply one — appropriate for a workflow where students are registered in real time rather than backdated in batches.
- **`Gender`**: a single fixed character, defaulting to `'M'` for this particular (all-male) college context, demonstrating that a default value is simply overridden whenever the inserting statement explicitly supplies its own value for that column.

After creating the table, its structure is inspected using **`DESCRIBE table_name`** (or by browsing to it directly in the schema panel), which lists every column, its data type, and whether it's nullable — useful for reacquainting yourself with a table you didn't create recently, or don't otherwise have documentation for.

Rows are then added with an `INSERT` statement of the general shape:

```sql
INSERT INTO Students VALUES (105, 'Ahmed Ali', 3.75, '10-SEP-2023', 'M');
```

And all rows are retrieved back with:

```sql
SELECT * FROM Students;
```

The walkthrough also demonstrates **partial inserts** — providing values for only some columns by explicitly naming them (`INSERT INTO Students (Student_ID, First_Name, GPA) VALUES (...)`), which lets `NOT NULL` and `DEFAULT` rules kick in for the columns left out: any omitted column with a default (like `Registration_Date` or `Gender`) is auto-filled rather than left blank, while any omitted column marked `NOT NULL` (with no default) would cause the insert to fail if left out entirely.

## 6. Interview Questions & Key Concepts

### Fundamentals

**Q1. What are the main categories of SQL commands, and what does each one do?**
*DDL (Data Definition Language) defines or alters object structure — e.g., `CREATE`, `ALTER`, `DROP`; DML (Data Manipulation Language) changes data — `INSERT`, `UPDATE`, `DELETE`; DQL (Data Query Language) retrieves data — `SELECT`; DCL (Data Control Language) manages permissions — `GRANT`, `REVOKE`; and TCL (Transaction Control Language) manages multi-step transactions — `COMMIT`, `ROLLBACK`, `SAVEPOINT`. Some references group `SELECT` under DML instead of a separate DQL category, but the functional split — structure, data, retrieval, access, and transactions — is the standard way SQL is categorized today.*

**Q2. What's the difference between `CHAR` and `VARCHAR2` (or `VARCHAR`), and when would you choose one over the other?**
*`CHAR` is fixed-length — it always reserves the full declared size regardless of the actual data's length, which can waste storage. `VARCHAR2`/`VARCHAR` is variable-length — it only uses as much space as the data actually needs, up to the declared maximum. In practice, `VARCHAR2`/`VARCHAR` is preferred for most text columns (like names or addresses) precisely because it avoids wasted, fixed reserved space; `CHAR` remains useful for genuinely fixed-length values, such as a two-letter country code.*

**Q3. What are the three stages a SQL statement goes through before returning results?**
*Parsing (checking the statement's syntax and confirming referenced objects exist), optimization (the database's query optimizer selecting the most efficient execution plan among the available options, based on factors like data volume and location), and execution (actually carrying out the chosen plan and returning the final result set).*

### Data Modeling & Constraints

**Q4. What's the difference between the `NOT NULL` and `DEFAULT` constraints, and can they be used together?**
*`NOT NULL` requires that a value be supplied for that column — an insert that leaves it out is rejected. `DEFAULT` supplies an automatic value when none is given, so the column is effectively never left blank without the insert needing to fail. They're often combined in modern practice: a `DEFAULT` value guarantees the column is populated automatically, while `NOT NULL` still guards against explicitly inserting a null value on top of it.*

**Q5. Why is it good practice to split composite data (like a full name or an address) into separate columns rather than storing it as one block of text?**
*Splitting composite fields (first/last name, or city/street/house-number for an address) makes searching, filtering, and sorting far more reliable — a query for "everyone in Riyadh" is trivial against a dedicated city column, but unreliable against free-text addresses where people entered the city in inconsistent positions or formats. This is also foundational to normalization, specifically to keeping data atomic as required by First Normal Form.*

**Q6. What happens if you insert a decimal value with more digits after the decimal point than a `NUMBER` column's declared scale allows — versus more digits before the decimal point than its precision allows?**
*Oracle rounds the value to fit the declared scale (the number of digits after the decimal point) rather than rejecting it — e.g., inserting `3.7857` into a `NUMBER(3,2)` column stores `3.79`. However, if the whole-number part exceeds what the declared precision allows for (e.g., a two-digit whole number where only one digit is permitted), the insert is rejected outright, since that would violate the column's total digit capacity.*

### Business Logic & Practical Use

**Q7. Why does a bank transfer need to be handled as a transaction (TCL) rather than as separate, independent SQL statements?**
*A transfer involves multiple dependent steps — checking the balance, debiting one account, crediting another, logging the result — and all of them need to succeed together or not at all. TCL commands (`COMMIT`/`ROLLBACK`, with `SAVEPOINT` for intermediate checkpoints) let the database guarantee this all-or-nothing behavior; industry practice ties this to the broader ACID guarantees (Atomicity, Consistency, Isolation, Durability) that relational databases provide for transactions.*

**Q8. Why does the SQL query optimizer matter for real-world performance, and what kinds of factors does it weigh?**
*A single query can often be executed in more than one way internally (e.g., in what order to join multiple tables, or which stored copy of the data to read from if it's mirrored across locations); the optimizer picks the plan expected to run fastest based on factors like expected result size and data location. Getting this right is what keeps queries fast as data volume grows — a concept the industry now extends further with tools like `EXPLAIN PLAN` and query execution statistics to analyze and tune an optimizer's chosen plan.*

**Q9. What's the practical purpose of `DESCRIBE` (or an equivalent schema-inspection command/tool) when working with an unfamiliar table?**
*It quickly surfaces a table's structure — column names, data types, and nullability — without needing separate documentation, which is especially useful when working with a table you didn't create yourself, or returning to a project after time away. Most modern database tools also expose the same information through a visual schema browser as an alternative to typing the command.*

**Q10. What's the difference between granting a user a privilege and defining a table's structure — why are these kept in separate SQL command categories?**
*Granting/revoking privileges (`GRANT`/`REVOKE`, DCL) governs *who* can act on the database and in what ways, entirely separate from *what* the database's objects look like (DDL) or *what data* they hold (DML). Keeping access control as its own category allows fine-grained, auditable, and revocable permissions — for example, granting a specific user `SELECT`-only access to a table without giving them the ability to modify its structure or its data.*

---

*Approximate word count: 2,350 words.*
