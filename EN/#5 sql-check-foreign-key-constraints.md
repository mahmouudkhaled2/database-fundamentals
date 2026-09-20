# CHECK and FOREIGN KEY Constraints in SQL

This lesson continues a series on SQL **Constraints**, picking up after `NOT NULL`, `UNIQUE`, and `PRIMARY KEY` to cover the **CHECK** constraint and the **FOREIGN KEY** constraint in depth, using an Oracle Live SQL walkthrough.

## Key Takeaways

- A **CHECK constraint** validates a rule *within a single row of a single table* (e.g., "end date must be after start date"), while a **FOREIGN KEY constraint** validates that a value in a child table actually *exists* in a parent table's key column.
- Both constraints can be written **column-level** (right next to the column) or **table-level** (grouped after all columns are declared) — table-level is required whenever a constraint spans more than one column.
- A composite **PRIMARY KEY** is often necessary when a single column would repeat across rows (e.g., an employee who moves through several jobs needs `employee_id + start_date` together as the key, not `employee_id` alone).

## 1. What is a CHECK / FOREIGN KEY Constraint?

A **CHECK constraint** is a rule that inspects the data being inserted or updated and confirms it complies with a boolean condition — for example, that a salary is never below a minimum value, or that a grade falls between 0 and 5. If the condition evaluates to false, the row is rejected.

A **FOREIGN KEY constraint** is what actually implements a **relationship** between two tables. It forces a column (or set of columns) in a "child" table to only contain values that already exist in the **primary key** (or a unique column) of a "parent" table. In the video's analogy, it works like a birth certificate listing a father's national ID — the record only makes sense if that ID genuinely exists in the parent registry.

## 2. Technical Flow (The "Hops")

1. **Define the rule at design time.** The developer writes a `CHECK` or `FOREIGN KEY` clause either next to a column (column-level) or after all columns (table-level).
2. **A DML statement arrives.** A user attempts to `INSERT` or `UPDATE` a row.
3. **Constraint evaluation kicks in automatically.** For a `CHECK` constraint, the database evaluates the boolean expression using **only values from that same row**.
4. **For a FOREIGN KEY, the database looks up the parent table.** It searches the referenced column in the parent table for a matching value.
5. **Pass or fail:**
   - If a `CHECK` condition is false, or a foreign key value has no match in the parent table, Oracle raises an error (the video shows a real `ORA` "parent key not found" and check-constraint-violation error) and the transaction is rejected.
   - If everything is valid, the row is committed.
6. **Special case — NULLs are allowed.** A foreign key column can be left `NULL` (meaning "not yet assigned," e.g., an employee not yet placed in a department); an empty value is not checked against the parent table.

## 3. Why Do We Need These Constraints?

- **Enforce business rules automatically**, so bad data (like a negative salary or a resignation date before a hire date) can never be recorded, without the developer having to remember to check it every time.
- **Guarantee referential integrity** — a department ID stored against an employee is guaranteed to be a department that actually exists, so reports and joins never point to "ghost" records.
- **Reduce reliance on manual/application-side checks**, since the rule lives once in the schema and applies to every insert or update, from any application or user.
- **Make relationships between entities concrete.** In ER modeling, entities are connected by relationships; the FOREIGN KEY is the mechanism that physically implements that relationship between two tables.
- **Save long-term maintenance effort** — once written, the rule runs forever without needing to be re-checked by hand.

## 4. Key Comparisons

### Why can't we just validate data in the application instead of the database?

1. **Multiple entry points bypass application logic.** Data can be inserted directly via other tools, scripts, or a different application entirely — a check written only in one app's code won't protect the table from those paths, but a database-level constraint always applies.
2. **A single source of truth.** Putting the rule in the database means every consumer of that table — reports, other apps, ad hoc queries — inherits the same guarantee, instead of every team re-implementing the same validation.
3. **Application checks can't fully guarantee referential integrity.** Confirming that a foreign-key value truly exists in another table, at the exact moment of the transaction, is something the database engine is positioned to do reliably and atomically; duplicating that logic in application code is error-prone and can race with concurrent writes.

(In practice, most systems use *both*: friendly, fast validation in the application UI, and constraints in the database as the non-negotiable final guarantee.)

## 5. Deep Dive: Oracle Syntax and Design Details

### CHECK constraint — three worked examples

Using an `employee_job_history` table (`employee_id`, `start_date`, `job_id`, `end_date`, `department_id`, `salary`), the instructor builds:

- A **table-level** CHECK ensuring the end date comes after the start date:
```sql
CONSTRAINT emp_job_history_end_start_check
  CHECK (end_date > start_date)
```
- A **column-level** CHECK restricting salary to a specific range:
```sql
salary NUMBER(6,2) CONSTRAINT salary_check CHECK (salary BETWEEN 100 AND 200)
```

Important limitations called out explicitly in the lesson (and confirmed by current Oracle documentation):
- A `CHECK` condition **must only reference columns of the same row** — it cannot compare a value in one row against a value in another row, and it cannot reference a column in a different table.
- A `CHECK` condition **cannot use non-deterministic functions** such as `SYSDATE`, `USER`, `UID`, `USERENV`, sequence values (`CURRVAL`/`NEXTVAL`), or pseudo-columns like `ROWNUM`/`LEVEL` — the same row's values must be able to be evaluated deterministically at insert/update time.
- There is **no limit** to how many `CHECK` constraints can be defined on a single column.

### Composite PRIMARY KEY example

The lesson stresses that `employee_id` alone cannot be the primary key of a job-history table, because one employee can pass through several jobs over time (clerk → secretary → office manager, etc.), which would repeat the same `employee_id`. The fix is a **composite primary key** of `employee_id + start_date`, since a single employee cannot start two different jobs on the same date. This illustrates that a key is chosen based on the actual business scenario, not by default habit.

### FOREIGN KEY constraint — syntax and requirements

Two equivalent ways to declare it, shown against a `department_id` column referencing a `department` table:

**Column-level (shorthand), directly after the column:**
```sql
department_id NUMBER REFERENCES department(department_id)
```

**Table-level (formal), naming the constraint explicitly:**
```sql
CONSTRAINT emp_job_history_dept_id_fk
  FOREIGN KEY (department_id)
  REFERENCES department (department_id)
```

Key requirements and observations from the walkthrough:
- The **parent table's referenced column must be a PRIMARY KEY or at least a UNIQUE column** — otherwise Oracle has nothing reliable to link against.
- **Duplicate values are normal and expected in the foreign key column** (many employees can share the same `department_id`), unlike a primary key, which can never repeat.
- **NULL is allowed** in a foreign key column — it simply means the relationship hasn't been assigned yet (e.g., a department that exists on paper but has no staff yet, or an employee not yet placed in a department).
- **A parent row can exist with zero matching child rows** (a department created but with no employees) — that's fine; what's invalid is the reverse: a child value with no matching parent.
- If an insert references a parent value that doesn't exist, Oracle rejects it with a **"parent key not found"** error — demonstrated live by inserting `department_id = 30` when only `10` and `20` exist in the parent table.
- **Clear, descriptive naming matters.** The instructor recommends a convention like `table_column_fk` (e.g., `dept_id_fk`) so that any teammate reading the constraint name instantly understands its purpose without digging into the code.

## 6. Interview Questions & Key Concepts

### Fundamentals

1. **What is the difference between a CHECK constraint and a FOREIGN KEY constraint?**
   *A CHECK constraint validates a condition using only the values within the same row of the same table (e.g., a range or a comparison between two columns of that row). A FOREIGN KEY constraint validates that a value exists in another table's primary/unique key, enforcing a relationship between two tables.*

2. **Can a CHECK constraint reference a column from a different table?**
   *No. A CHECK constraint can only compare columns within the same row; referencing another table's column is not permitted, which is exactly what a FOREIGN KEY is for.*

3. **Can a foreign key column contain NULL values?**
   *Yes. Unless the column also has a NOT NULL constraint, a foreign key may be NULL, meaning the relationship simply hasn't been established yet — NULL values are not checked against the parent table.*

4. **What functions or values are disallowed inside a CHECK constraint expression?**
   *Non-deterministic or session-dependent elements such as SYSDATE, USER, UID, USERENV, sequence pseudo-columns (CURRVAL/NEXTVAL), and ROWNUM/LEVEL cannot be used, because the condition must be evaluable purely from the row's own column values.*

### Architecture & Security

5. **What must be true about the column a FOREIGN KEY references in the parent table?**
   *It must be a PRIMARY KEY or at least a UNIQUE-constrained column in the parent table; a foreign key cannot reference an arbitrary, non-unique column.*

6. **Why might a database designer use a composite primary key instead of a single-column key?**
   *When no single column can uniquely identify a row on its own — for example, in a job-history table where the same employee ID legitimately repeats across multiple job records, so employee_id + start_date together form the unique identifier.*

7. **What real-world error occurs when you try to insert a foreign key value that has no match in the parent table, and how does the database prevent it in general?**
   *The database raises an integrity/"parent key not found" error and rejects the transaction. More broadly, referential-integrity enforcement can happen either immediately at insert/update time (the default) or be deferred to commit time in databases that support deferrable constraints.*

8. **What's the difference between a foreign key value repeating and a primary key value repeating?**
   *Repeating values in a foreign key column are completely normal (many child rows can point to the same parent), while a primary key must always be unique — a repeated primary key value is a data-integrity violation.*

### Business Logic

9. **Why enforce data-validation rules (like salary ranges or valid date ordering) at the database level instead of only in the application?**
   *Because data can enter a table through multiple paths — different applications, scripts, or direct access — and a rule defined once in the schema applies uniformly to all of them, whereas application-only validation can be bypassed or duplicated inconsistently across systems.*

10. **How does a FOREIGN KEY constraint model a real-world business relationship, such as an employee belonging to a department?**
    *It guarantees that any department referenced by an employee record genuinely exists in the department table, mirroring the real-world rule that an employee cannot belong to a department that doesn't exist — this is the mechanism that turns an ER-diagram relationship into an enforced database rule.*

11. **What happens to child records when a parent record referenced by a FOREIGN KEY is deleted, and how can that behavior be controlled?**
    *By default, deleting a referenced parent row is blocked if matching child rows exist. This behavior can be changed with options such as `ON DELETE CASCADE` (deletes matching child rows automatically) or `ON DELETE SET NULL` (clears the foreign key in child rows), which should be chosen deliberately based on the business need.*

12. **Should a department or lookup table be allowed to contain rows with no matching child records yet?**
    *Yes — a parent table can have valid rows that no child table references yet (e.g., a newly created department with no employees assigned); this is normal and does not violate referential integrity. The violation only runs the other direction: a child row pointing to a parent value that doesn't exist.*

---

*Approximate word count: 1,750 words.*
