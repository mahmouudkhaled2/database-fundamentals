# Flat-File vs. Relational Databases: Keys, Design Rules, and Getting Started with SQL

This session picks up where the intro to databases left off, explaining *why* the industry moved away from storing everything in a single flat file toward the relational model, then walks through the rules for designing proper tables — including primary and foreign keys — before doing a live, hands-on first pass at writing SQL in Oracle Live SQL.

## Key Takeaways

- **Flat-file storage** (one big table holding everything) is simple to read and cheap to run, but it causes **data redundancy, inconsistency, weak security, and poor scalability** once more than one department or system needs to use the same data.
- The **relational model** fixes these problems by splitting data across multiple linked tables, using a **primary key** to uniquely identify each row and a **foreign key** to reference related data in another table — at the cost of needing joins/queries to reassemble a "full picture" of an entity.
- Every table needs one column (or set of columns) chosen as its **primary key** — it must be **unique** and **NOT NULL** — while a table can have several **candidate keys** (unique-but-not-chosen columns) that could have served the same purpose.

## 1. What Is the Relational Model (as Opposed to a Flat File)?

A **flat file** database is, in effect, a **single table** ("single table") that stores *all* the data for a subject in one place, with no separation of concerns and no formal structure connecting different kinds of data — the instructor stresses that "flat" specifically means **no structure**: everything about a customer, for instance, sits in one wide row rather than being organized into related, purpose-built tables.

A **relational database** ("relational" comes from the idea of **relations/relationships**) instead stores data as **multiple tables**, each focused on a single subject, connected to each other through shared columns. Rather than cramming every fact about a customer — their multiple addresses, multiple phone numbers, their orders, and so on — into one record, a relational design splits these into separate tables (Customer, Address, Phone, Order) and links them, because that mirrors how the data actually relates in the real world.

## 2. Technical Flow: From a Single Flat File to a Working Relational Table

The lecture moves through a practical progression, from *why* flat files fall short to *actually creating* a table with a primary key in a live SQL environment:

1. **Start from the old approach — a flat file.** All records for an entity (e.g., customers) are stored in one plain-text-style table, one row per customer, with every attribute about that customer packed into the same row.
2. **Recognize the model's inherent limitations** as data grows and gets shared: no access control, duplicate copies drifting out of sync, no protection against re-entering the same data twice, and no ability to safely share the file across departments.
3. **Split the data into multiple related tables** (the relational approach): each table represents one entity, and tables reference each other through shared columns instead of duplicating full records.
4. **Assign a primary key to every table** — a column (or combination of columns) whose value is guaranteed unique and never blank, used to pinpoint one exact row.
5. **Evaluate candidate keys, then choose the primary key** — a table often has several columns that could uniquely identify a row (ID number, email, national ID, driver's license number); the designer picks the one best suited to *that particular system's* purpose (a university uses the student ID; the traffic department uses the driver's license number; civil records use the national ID).
6. **Link tables together with foreign keys** — a column in one table (e.g., `department_id` in an Employees table) that stores the primary-key value from another table (the Departments table), letting you "hop" from one table to related details in the other without duplicating that data.
7. **Apply table- and column-level design rules** — consistent naming conventions, one value per cell, consistent data types per column, and other constraints covered in the deep-dive section below.
8. **Translate the design into actual SQL** — create an account on an online SQL tool (Oracle Live SQL, in this walkthrough), write a `CREATE TABLE` statement defining columns and data types, run it, and confirm the table now appears in the schema.
9. **Manage and iterate on scripts** — save scripts with descriptive names and descriptions, edit or delete individual statements, revisit prior sessions, and adjust settings like the maximum number of rows returned by a query.

## 3. Why Do We Need the Relational Model?

Splitting data across related tables (instead of keeping one flat file) solves a specific set of real, recurring problems:

- **Reduced redundancy** — shared or repeated information (like a department's details) is stored once, in its own table, and simply referenced elsewhere instead of being copied into every related row.
- **Consistency** — because data lives in one authoritative place, an update (e.g., a customer's new address) is reflected everywhere at once, instead of some copies getting updated while others silently go stale.
- **Data integrity through constraints** — a relational table lets you enforce rules directly on the data: a value range (e.g., GPA between 0 and 5), a required field (`NOT NULL`), a mandatory match against another table (you can't enroll a student in a course ID that doesn't exist in the Courses table), and uniqueness (no duplicate student IDs).
- **Confidentiality / access control** — because data is split by subject into different tables, access can be restricted at a finer grain (e.g., only Finance staff can see a Salary table), instead of everyone who can open "the file" seeing everything, including sensitive fields like salaries or personal addresses.
- **Better performance and scalability at large volumes** — retrieving, filtering, and maintaining data stays efficient even as record counts grow into the millions, which a single giant flat file struggles with.
- **Safer duplicate prevention** — the DBMS can automatically flag or reject an attempt to insert data that already exists (e.g., an email already registered), something a flat file has no built-in way to check.

## 4. Key Comparisons: Flat-File Storage vs. the Relational Model

| | Flat File (single table) | Relational Model (multiple linked tables) |
|---|---|---|
| Structure | None — one wide table holds everything | Structured — data split by entity across many tables |
| Reading/understanding data | Easy at a glance, since it's all in one place | May require joining several tables to see the full picture |
| Setup effort | Minimal hardware/software required | Needs a full DBMS to manage relationships and constraints |
| Redundancy | High — the same data can be copied into multiple files/departments | Low — data is centralized and referenced, not duplicated |
| Consistency | Poor — updates in one copy don't propagate to others | Strong — single source of truth per entity |
| Security | Weak — anyone with the file sees everything | Granular — access can be restricted table-by-table |
| Scalability | Struggles as record volume grows | Designed to stay efficient at large scale |

### Why can't we just use flat files instead of a relational database?

1. **No access control.** A flat file is, in the end, just a file — there's no mechanism to say "this person can see it, that person can't," since that requires an actual DBMS layer managing user permissions.
2. **Sensitive data exposure.** Because everything about an entity sits in one record, giving someone access to "the customer data" for a legitimate reason (like billing) also hands them unrelated sensitive fields (like salaries or personal contact details) they shouldn't see.
3. **Data inconsistency across duplicated copies.** When the same information needs to live in several departments (sales, billing, support), each keeps its own copy of the flat file — and when a customer updates one detail, only the copy that received the update reflects it, while the rest silently go out of date, causing real operational errors (like a bill mailed to an old address).
4. **No duplicate-prevention.** A flat file has no built-in way to warn you that a record already exists, so the same data can end up entered multiple times without you knowing, unlike a relational database, which can reject or flag it automatically.
5. **No safe way to share the data.** Sharing a flat file means sharing *everything* in it at once, with no partial or secure sharing option — you can't hand out only the fields a given person actually needs.
6. **Poor performance at scale.** As the number of records grows into the hundreds of thousands or millions, a single flat file becomes slow and unwieldy to search, sort, and maintain.

*(A brief accuracy note: this is the practical, intro-level framing the lecture uses. In formal database theory the same points are usually summarized as the goals of **normalization** — eliminating redundancy and update/insert/delete anomalies — which remains the standard way this trade-off is taught today.)*

## 5. Deep Dive: Keys, Table Design Rules, and a First Look at Oracle Live SQL

### Candidate Keys and the Primary Key

A **unique/key column** is any column whose values never repeat — a student's university ID, email, national ID, or driver's license number could all serve this purpose for a Students table, and each one that qualifies is called a **candidate key**. A table can have multiple candidate keys, but the designer selects **one** to actually use as the table's working identifier, based on the system's purpose: a university system typically keys on the university ID, civil records key on the national ID, and the traffic department keys on the driver's license number — different organizations can legitimately choose different "primary" identifiers for the same person.

The column chosen this way becomes the **primary key**, and it must satisfy two conditions:

- **Unique** — its value can never repeat across rows in the table.
- **NOT NULL** — it can never be left empty; every row must have one.

Not every unique column qualifies as a good primary key candidate for this reason — the instructor notes that a driver's license number is unique but *can* be null (not everyone has one), so it fails the "not null" requirement for a general population table, whereas a university ID (assigned to every enrolled student) satisfies both conditions.

### Practical Tip: Prefer Numbers Over Text for Keys

When a choice is available, the lecture recommends preferring **numeric** keys over text-based ones for two practical reasons:

- Numbers are far less prone to entry mistakes than text — the same name can be typed with different spellings ("Mohammed" vs. "Muhammad," a doubled letter, etc.), which breaks exact-match lookups.
- Searches are **case-sensitive** at the character level, because every character maps to a distinct **ASCII code**, and uppercase and lowercase versions of the same letter have different codes. A search for "Dana" (capitalized) will not match a stored value of "dana" (lowercase), so relying on text as a key introduces a class of matching bugs that numeric keys avoid entirely.

### Foreign Keys

A **foreign key** is a column added to one table specifically to reference the primary key of another table, establishing the link between them — for example, storing a `department_id` inside the Employees table so you can trace any employee back to their department's full details in the Departments table, without copying the whole department record into every employee row. The name "foreign" reflects that this column doesn't natively belong to the table it sits in — it's *borrowed* from another table purely to create the relationship.

A key distinction the lecture draws out: a value that must be unique as a **primary key** in its own table (e.g., department 10 appears only once in the Departments table) is expected to **repeat** as a **foreign key** wherever it's referenced (many employees can share `department_id = 10`) — that repetition is normal and correct, not a violation of uniqueness rules.

### Table and Column Design Rules

The lecture lays out a set of conventions and constraints to follow when designing relational tables:

- **Table names are conventionally plural** (e.g., `Employees`, `Students`) — not a hard requirement, but a widely followed industry convention that new team members are expected to pick up when joining a company's codebase.
- **No two tables in the same database can share a name** (though the same name can be reused across different databases).
- **Every table must have a primary key**, so each row can be distinguished from every other row.
- **No two columns in the same table can share a name** (though the same column name can appear in different tables — e.g., `id` in several tables is fine).
- **A single cell can hold only one value** — a "phone" column, for instance, can't store three phone numbers separated by commas; each value needs its own column or its own row in a related table.
- **A column must hold one consistent data type** — a "name" column can't mix text in some rows with numbers in others.
- **Row and column order carries no meaning for storage** — the physical order you insert rows in, or the order columns are listed in, doesn't matter; retrieval order for a report is instead controlled at query time (the lecture flags `ORDER BY` as the clause used later for this, sorting ascending or descending as needed).

### First Steps in Oracle Live SQL

The walkthrough uses **Oracle Live SQL**, a free, browser-based tool for writing and running SQL without installing anything locally (an alternative to installing Oracle Express Edition). After creating a free account (email, password meeting Oracle's complexity rules, country, and basic contact details) and signing in, the key parts of the interface covered are:

- The **SQL Worksheet** — where statements are typed and executed by clicking **Run** (the highlighted statement is the one that gets executed).
- The **Max Rows** setting, found under Actions, controlling how many result rows are displayed (the lecture increases it from the 50-row default for larger result sets).
- **Save Script** — saves the current set of statements under a name and optional description, and a visibility setting (private vs. shared/public with a link).
- **My Scripts** and **My Sessions** — lets you reopen previously saved scripts or revisit an entire earlier working session, including statements that were run before.
- **The Schema panel** — shows both a set of ready-made sample tables provided by Oracle for practice, and any tables you create yourself, so you can inspect a table's structure and stored data at any time.
- **Tutorials** — a built-in, searchable reference for SQL command syntax.

As a first concrete example, the lecture creates a simple table with a `CREATE TABLE` statement defining an `Employee_ID` (numeric, fixed length) and a `First_Name` (character-based) column, runs it, confirms the new table shows up in the schema list, and demonstrates editing the saved script (deleting an unwanted statement, then re-running to confirm the change took effect).

## 6. Interview Questions & Key Concepts

### Fundamentals

**Q1. What's the core difference between a flat-file database and a relational database?**
*A flat-file database stores all data for a subject in a single, unstructured table, while a relational database splits data across multiple linked tables, each dedicated to one entity, connected through shared key columns. The relational approach trades some simplicity for major gains in consistency, security, and scalability as data grows.*

**Q2. What problems does the relational model solve that flat files don't?**
*Primarily data redundancy (the same facts duplicated across copies), data inconsistency (updates not propagating to every copy), weak security (no way to restrict access to specific fields), and poor performance/scalability at large record volumes. These map closely to the goals of formal database normalization: eliminating redundancy and the insert/update/delete anomalies that come with it.*

**Q3. What is a primary key, and what two properties must it always have?**
*A primary key is the column (or combination of columns) chosen to uniquely identify each row in a table. It must be unique (no duplicate values) and NOT NULL (every row must have a value) — a column that's unique but can legitimately be empty, like an optional ID number, is disqualified from being the primary key even though it may still be a valid candidate key.*

**Q4. What is the difference between a candidate key and a primary key?**
*A candidate key is any column (or set of columns) that could uniquely identify a row — a table can have several (ID number, email, national ID). The primary key is the one candidate key actually selected as the table's official identifier, chosen based on what best fits the system's purpose.*

### Data Modeling & Keys

**Q5. What is a foreign key, and how does it differ in behavior from a primary key?**
*A foreign key is a column that stores the primary-key value from another table, creating a link between the two. Unlike a primary key, a foreign key's values are expected to repeat — many rows in the referencing table can legitimately point to the same row in the referenced table (e.g., many employees sharing the same department ID). Current best practice also has the database enforce this link as a formal constraint (referential integrity), so a foreign key value must correspond to an existing primary key in the referenced table.*

**Q6. Why would a database designer prefer a numeric primary key over a text-based one?**
*Numeric values are far less error-prone to enter and match exactly than text, which is vulnerable to spelling variations and, critically, case sensitivity — since every character (uppercase or lowercase) maps to a distinct underlying character code, "Dana" and "dana" will not match in an exact search even though they look the same to a human reader.*

**Q7. What is a composite key, and when is it used?**
*A composite key is a primary (or candidate) key made from two or more columns combined, used when no single column is unique on its own — commonly in linking/junction tables, such as a student-course enrollment table where the pair (student_id, course_id) together is unique even though neither column is unique alone. This concept extends naturally from the single-column keys covered in the lecture and is standard in relational design today.*

**Q8. Why must every column in a table hold only one value per cell, and only one data type?**
*Storing multiple values in one cell (e.g., three phone numbers separated by commas) breaks the ability to search, sort, filter, or join on that data reliably, and mixing data types in a column breaks any calculation or comparison performed on it. Both rules are foundational to keeping a table in proper relational (specifically, First Normal Form) shape.*

### SQL & Practical Use

**Q9. What does a `CREATE TABLE` statement need to define at minimum?**
*The table name (conventionally plural) and, for each column, a name and a data type (and, where relevant, a size or precision, such as a fixed character length or a number of decimal digits) — along with any constraints like `NOT NULL` or a primary key designation.*

**Q10. Why is it good practice to add descriptions when saving SQL scripts?**
*Descriptions make it much easier to find and reuse past work later — without them, a saved script full of similarly named files becomes difficult to navigate, especially over a long course or project, whereas a short note on what a script covers (e.g., "creating the Employees table") lets you return to exactly the right work quickly.*

**Q11. What does it mean that "row and column order doesn't matter" in a relational table, and how do you control display order instead?**
*The physical order in which rows or columns were entered has no bearing on how the data is logically stored or interpreted — a relational table is a set of rows, not a sequence. To control the order data is *returned* in in a query result, you use the SQL `ORDER BY` clause (ascending or descending), rather than relying on insertion order.*

**Q12. How does a relational database prevent inserting a "child" record that points to a non-existent "parent" — for example, enrolling a student in a course ID that doesn't exist?**
*By enforcing the foreign key as a formal constraint referencing the parent table's primary key: the DBMS rejects any insert or update whose foreign key value doesn't already exist as a primary key value in the referenced table, which is the standard mechanism (referential integrity) relational databases use to keep linked data consistent.*

---

*Approximate word count: 2,200 words.*
