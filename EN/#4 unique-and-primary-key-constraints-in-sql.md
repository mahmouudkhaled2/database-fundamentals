# UNIQUE and PRIMARY KEY Constraints: Column-Level vs. Table-Level Syntax

This session drills into two of SQL's core constraint types — **UNIQUE** and **PRIMARY KEY** — showing the different ways they can be written, why naming a constraint matters, and how to enforce uniqueness across more than one column at a time.

## Key Takeaways

- A **constraint** is a rule applied to a column (or columns) so that stored data stays consistent with real-world business rules — a **UNIQUE** constraint blocks duplicate values, while a **PRIMARY KEY** combines uniqueness with a mandatory (`NOT NULL`) requirement to serve as the table's official row identifier.
- Constraints can be written at the **column level** (right next to the column definition) or the **table level** (grouped together after all columns are listed); the table-level style is the only option when a constraint must apply to **more than one column at once** — a **composite constraint**.
- Always give constraints an explicit **name**. Skipping it lets the database generate an unhelpful system name, which makes error messages far harder to trace back to the actual rule that was violated.

## 1. What Are UNIQUE and PRIMARY KEY Constraints?

A **constraint** in a relational database is a rule or condition applied to a column that incoming data must satisfy — it exists to keep stored data aligned with the real business rules behind it (for example, a GPA must fall between 0 and 5, or a salary must be at least a certain minimum).

Two specific constraint types are the focus here:

- **UNIQUE** — guarantees that no two rows can share the same value in that column (or combination of columns). It's used for anything that should never repeat but isn't necessarily the table's main identifier — an email address, a national ID number, a phone number.
- **PRIMARY KEY** — the column (or combination of columns) chosen as the table's official row identifier. It automatically carries **both** the uniqueness requirement **and** a `NOT NULL` requirement, so it can never be duplicated and can never be left empty.

## 2. Technical Flow: From a "Quick" Constraint to a Properly Named One

The lecture walks through building a `Workers` table and iterating on how its constraints are written, exposing the pitfalls of shortcuts along the way:

1. **Start with the shorthand (column-level, unnamed) syntax** — write `UNIQUE` directly next to a column's definition, right after its data type, for each column that needs it (e.g., `Worker_ID`, `Email`, `Mobile_Number`).
2. **Hit a real error** — attempting to insert a row that violates one of several unnamed `UNIQUE` constraints produces an error message that only references an auto-generated system constraint name, giving no clue which column actually caused the problem.
3. **Recognize the fix: give every constraint an explicit name.** Rewriting the same shorthand syntax but adding a name (e.g., `Worker_ID_Unique`) right before the `UNIQUE` keyword makes every future violation message point directly at a human-readable name tied to the specific rule that failed.
4. **Learn the formal, table-level alternative.** Instead of scattering constraint definitions next to each column, list all the column definitions first, then declare every constraint afterward as its own named `CONSTRAINT constraint_name UNIQUE (column_name)` clause — considered the more standard, "formal" style, especially valuable once a table has many columns and constraints.
5. **Apply the same idea to PRIMARY KEY.** Instead of marking a column `UNIQUE`, mark it (or reference it in a table-level clause) as `PRIMARY KEY`, which automatically implies both uniqueness and `NOT NULL` — attempting to also declare it separately as `UNIQUE` or leave it nullable becomes redundant or contradictory.
6. **Handle the case of enforcing uniqueness across two columns together (a composite constraint).** When two separate columns need to stay unique only as a *pair* — not individually — the column-level syntax can't express that; only the table-level syntax can, by listing both column names together inside one constraint's parentheses.
7. **Verify the resulting behavior with real inserts** — testing that a single-column violation is correctly rejected, that a composite constraint accepts a repeated value in one column as long as the *pair* is still unique, and that the primary-key column correctly rejects both duplicate and empty values.

## 3. Why Do We Need Explicit, Well-Structured Constraints?

- **Enforcing real business rules automatically** — constraints let the database itself guarantee things like "no two workers share the same ID" or "salary can't be entered as a negative number," instead of relying on every application or user to get it right manually.
- **Fast, precise debugging** — a named constraint produces an error message that immediately tells you *which* rule was broken, saving significant time compared to a generic system-generated name that requires extra investigation to trace back to the actual column and rule involved.
- **Guaranteeing a reliable row identifier** — the primary key constraint is what makes it possible to always uniquely address one specific row, which every later operation (updates, deletes, joins) depends on.
- **Modeling rules that span multiple columns** — some real business rules genuinely depend on a *combination* of fields (e.g., the same phone number is fine to reuse as long as it's paired with a different email); table-level composite constraints are what make expressing this possible at all.
- **Data type discipline avoiding overflow** — choosing the right type for numeric-looking data (like phone numbers or ID numbers) that will never be calculated on and may grow in digit-length over time avoids a very real failure mode: numeric overflow once a value exceeds what the chosen numeric type can hold.

## 4. Key Comparisons: UNIQUE vs. PRIMARY KEY

| | UNIQUE | PRIMARY KEY |
|---|---|---|
| Duplicate values | Never allowed | Never allowed |
| Empty (NULL) values | Allowed (a row can legitimately skip the field, e.g., a missing email or phone number) | Never allowed — every row must have a value |
| Number allowed per table | A table can have several UNIQUE constraints | A table can have only **one** PRIMARY KEY |
| Purpose | Marks a column that should never repeat but isn't necessarily the table's main identifier | Marks the table's single official row identifier |

*(Accuracy note: the lecture's core distinction — that a UNIQUE column can be left empty while a PRIMARY KEY cannot — matches standard SQL behavior across virtually all relational databases today, including the well-known detail that most engines still allow **NULL** in a UNIQUE column, and some even permit multiple NULLs in that same column, since NULL is treated as "unknown," not as a concrete duplicate value.)*

### Why can't we just use UNIQUE for everything, instead of having a separate PRIMARY KEY?

1. **UNIQUE alone doesn't guarantee a value exists.** A column that's only `UNIQUE` can still be left blank on some rows (e.g., not every worker has to have an email on file) — but a table absolutely needs a column that's *always* populated to reliably identify every row, which is exactly what `PRIMARY KEY`'s built-in `NOT NULL` guarantees.
2. **A table needs one clear, authoritative identifier.** If several columns are just marked `UNIQUE` with no single one designated as primary, there's no official, agreed-upon column for other tables to reference when linking data together (a role that becomes essential once foreign keys are introduced) — a table is limited to exactly one `PRIMARY KEY` specifically so that role stays unambiguous.
3. **Declaring the same column both ways is redundant.** Since `PRIMARY KEY` already implies uniqueness, adding a separate `UNIQUE` constraint on that same column adds nothing — the lecture demonstrates that Oracle will even flag this as a conflict/redundancy when both are declared on the same column.

## 5. Deep Dive: Writing Constraints Correctly in Oracle SQL

### Column-Level vs. Table-Level Syntax

**Column-level** syntax places the constraint keyword directly after a column's data type, inline with its definition:

```sql
Worker_ID NUMBER(6) NOT NULL UNIQUE
```

This reads easily for simple cases, but it has two real limitations demonstrated in the walkthrough:

- **Skipping a name produces unhelpful errors.** If you write `UNIQUE` with no explicit constraint name, the database silently generates its own internal name for the constraint. When a later insert violates it, the resulting error message only shows that generated name — giving no immediate indication of which column or rule actually failed, forcing extra investigation to track it down.
- **It can't express a rule spanning more than one column.** Column-level syntax only ever attaches to the one column it's written next to — there's no way to say "these two columns together must be unique" using this style.

**Table-level** syntax instead lists every column definition first, and then adds each constraint afterward as a separate, explicitly named clause:

```sql
CREATE TABLE Workers (
  Worker_ID     NUMBER(6),
  First_Name    VARCHAR2(20) NOT NULL,
  Last_Name     VARCHAR2(20) NOT NULL,
  Salary        NUMBER(8,2),
  Email         VARCHAR2(30),
  Mobile_Number VARCHAR2(16),
  CONSTRAINT Worker_ID_Unique UNIQUE (Worker_ID),
  CONSTRAINT Email_UQ UNIQUE (Email),
  CONSTRAINT Mobile_UQ UNIQUE (Mobile_Number)
);
```

This is described as the more **formal**, standard way to declare constraints — every rule is grouped together in one place, which becomes especially valuable once a table has many columns (the instructor notes having worked on a banking-related table with roughly 150 columns, where scattering constraints throughout the column list would make the table very hard to navigate).

### Naming Constraints (Why It Matters)

Rewriting the same shorthand syntax but giving the constraint an explicit name fixes the debugging problem directly:

```sql
Worker_ID NUMBER(6) NOT NULL CONSTRAINT Worker_ID_Unique UNIQUE
```

With this in place, violating the rule produces an error referencing `Worker_ID_Unique` by name — instantly telling you exactly which column and rule caused the failure, instead of a cryptic auto-generated identifier.

### Composite (Multi-Column) UNIQUE Constraints

Some rules only make sense when applied to a **combination** of columns rather than each column individually. The example used: a household might legitimately reuse the same phone number across two family members' records (e.g., two siblings sharing a parent's phone number), so making `Mobile_Number` unique on its own would incorrectly block that. What actually shouldn't repeat is the **pairing** of email and phone number together.

This is only expressible at the table level, by listing both columns inside one constraint:

```sql
CONSTRAINT Email_Mobile_UQ UNIQUE (Email, Mobile_Number)
```

With this constraint in place, the walkthrough confirms the expected behavior: reusing the same phone number is accepted as long as the email is different, and reusing the same email is accepted as long as the phone number is different — the constraint only blocks an insert where **both** values together already exist in some other row.

### PRIMARY KEY: Combining NOT NULL and UNIQUE

Marking `Worker_ID` as the table's `PRIMARY KEY` (whether inline or as a named table-level clause) automatically enforces both `NOT NULL` and uniqueness together — so a duplicate insert is rejected, and so is one that leaves the field empty, without needing to declare either rule separately. The lecture also demonstrates that Oracle detects and flags the redundancy if a column is separately marked `UNIQUE` and then also declared `PRIMARY KEY` — since the primary key constraint already subsumes it.

### A Note on Choosing NUMBER vs. Text for "Number-Looking" Data

While defining the `Mobile_Number` column, the lecture makes a practical data-typing point worth carrying forward: values like phone numbers, national ID numbers, and residency/ID numbers should generally **not** be stored as a numeric type (`NUMBER`), even though they look like numbers, for two reasons:

- **They're never used in arithmetic** — you'd never multiply or divide a phone number, so there's no benefit to storing it numerically.
- **They're prone to outgrowing a fixed numeric type as their digit-count changes over time** — a phone-number or ID-number format can be extended to more digits as a population grows, and if the value is stored as a fixed-size numeric type, this risks an **overflow**, where a value exceeds what that type can represent, causing errors. The example given: a 4-byte (32-bit) numeric type has a fixed maximum representable value, and a value entered beyond that limit cannot be stored correctly.

*(Accuracy note: the lecture's explanation of the exact numeric ceiling for a 4-byte signed integer is simplified — the precise, standard figure for a 32-bit signed integer's maximum value is 2,147,483,647, i.e., 2³¹ − 1, since one bit is reserved for the sign. The underlying advice — store non-calculated, potentially-growing "number-like" identifiers as text/character types rather than as a numeric type — is correct and remains standard practice today.)*

## 6. Interview Questions & Key Concepts

### Fundamentals

**Q1. What is the difference between a UNIQUE constraint and a PRIMARY KEY constraint?**
*Both guarantee that no two rows share the same value, but a UNIQUE constraint still allows NULL (an unpopulated value, and depending on the database, even more than one NULL is often permitted since NULL isn't treated as a duplicate), while PRIMARY KEY always implies NOT NULL as well, so it can never be left empty. A table can have multiple UNIQUE constraints, but only one PRIMARY KEY.*

**Q2. Why should every constraint be given an explicit name rather than left unnamed?**
*An unnamed constraint gets an auto-generated system name from the database, so when a rule is violated, the resulting error only references that opaque name, making it hard to tell which column or business rule actually failed. A named constraint (e.g., `Email_UQ`) produces an error that points directly at the intended rule, which is significantly faster to debug — a practice considered standard in professional schema design today.*

**Q3. What's the difference between a column-level and a table-level constraint definition?**
*Column-level syntax places the constraint inline, right after the column it applies to, and can only ever reference that one column. Table-level syntax lists all columns first, then declares constraints afterward as separate, named clauses — and it's the only way to define a constraint that spans multiple columns at once (a composite constraint). Table-level syntax is generally recommended for readability once a table has many columns and constraints.*

### Data Modeling & Keys

**Q4. What is a composite UNIQUE constraint, and when would you use one?**
*A composite UNIQUE constraint enforces uniqueness on the *combination* of two or more columns rather than on any single column alone — for example, requiring that the pairing of `Email` and `Mobile_Number` never repeats, even though either column individually may repeat across different rows. It's used whenever a real business rule genuinely depends on more than one field together, and per current SQL standards, a row is only considered a duplicate if every column in the composite key matches an existing row.*

**Q5. Why can a table have only one PRIMARY KEY but multiple UNIQUE constraints?**
*The primary key serves a specific structural role — the single, authoritative identifier used to reference a row unambiguously (including later, by foreign keys in other tables) — so allowing more than one would create ambiguity about which column actually identifies a row. UNIQUE constraints have no such structural role; they simply enforce non-duplication on whichever additional columns need it, so a table can have as many as its business rules require.*

**Q6. Why is it poor practice to store an identifier like a phone number or national ID as a numeric data type?**
*Numeric types are meant for values used in arithmetic and have a fixed maximum representable size; identifiers like phone numbers are never calculated on and are prone to needing more digits over time (e.g., as a country's population grows), which risks the stored value exceeding the numeric type's capacity — an overflow. Current best practice stores such identifiers as character/text types (e.g., VARCHAR2), which comfortably accommodate leading zeros and future digit growth without any overflow risk.*

### Business Logic & Practical Use

**Q7. A household shares one phone number across two family members' records. How would you design constraints so this doesn't get incorrectly blocked?**
*Making the phone-number column individually UNIQUE would wrongly prevent this legitimate case. Instead, a composite UNIQUE constraint on (Email, Phone_Number) allows the same phone number to appear more than once, as long as it's paired with a different email each time — only rejecting an insert where the exact same email-and-phone combination already exists.*

**Q8. What does an "overflow" error mean in the context of a numeric database column, and how do you prevent it?**
*It occurs when a value being inserted exceeds the maximum value the declared numeric type (with its specified precision) can represent, causing the insert to fail or the value to be truncated/rejected depending on the database. It's prevented by choosing a data type with sufficient precision for realistic future growth, or — for values that are never calculated on, like ID or phone numbers — by using a text-based type instead of a numeric one in the first place.*

**Q9. In practice, why might a database designer choose to declare all of a table's constraints at the end of the CREATE TABLE statement rather than inline with each column?**
*Grouping constraints together at the end makes them easier to review, audit, and maintain as a table grows — especially in real-world tables that can have dozens or even over a hundred columns, where scattering individual constraints throughout the column list would make it much harder to get a complete picture of every rule applied to the table at a glance. This table-level approach is also the only way to express constraints, like composite uniqueness, that span more than one column.*

---

*Approximate word count: 2,150 words.*
