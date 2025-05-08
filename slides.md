---
# theme: seriph
title: Databases — Comprehensive Review
class: text-center
info: |
  CST 363 • Spring 2025  
  Slidev version of the end‑of‑term review
drawings:
  persist: false
mdc: true
---

# Databases Review  
### From ER Modeling to NoSQL & Beyond

<small>A good database is *designed* — not stumbled into.</small>

---

## Session Road‑Map

<br>

<v-clicks>

- **Design** – ER modeling → normalization  
- **Relational Model & SQL** – algebra, DDL/DML, advanced queries  
- **Implementation** – storage, indexing, transactions  
- **Beyond Relational** – NoSQL, views, Redis, ORMs  

</v-clicks>

---

<br><br>

Exam is closed-book, but you can bring 3 physical pages 8.5 x 11 (both sides) or a 6 page Google Doc 

(You will submit this as a lab)


<br>

Section 1 Exam will be Tuesday May 13 at 4pm.


---
layout: section
---

# Part 1  
## Database Design
---

### Why Bother Designing?

<br>

<v-clicks>

- Hard to change schema later  
- Prevent redundancy & anomalies  
- Align with *real‑world* requirements  
- **Key Takeaway** Poor design ⇒ insert/update/delete anomalies  

</v-clicks>

---

### ER Modeling – Entities & Entity Sets

<br>

| Concept | Description | Example |
|---------|-------------|---------|
| **Entity** | Real‑world object | `student` |
| **Entity Set** | Collection of similar entities | all students |

<br>

<v-click>

> **Key Takeaway**  Remember: entity **set** becomes a table name.

</v-click>

---

### ER Modeling – Attributes

<br>

| Type | Example | Note |
|------|---------|------|
| Simple | `title : TEXT` | atomic |
| Composite | `address → {street, city, zip}` | break into parts |
| Multi‑valued | `phone {...}` | becomes new table |
| Nullable | `middle_name` | may be `NULL` |

---

### ER Modeling – Relationships

<br>

<v-clicks>

- **Relationship** = association (e.g., `advises`)  
- **Relationship set** = all occurrences  
- Each relationship has **roles** (advisor, advisee)  

</v-clicks>

---


### Mapping Cardinalities

<br>

| Cardinality | Example |
|-------------|---------|
| **1 : 1** | `id_card–student` |
| **1 : n** | `advisor–student` |
| **m : n** | `student–course (takes)` |


<br>

> Which cardinality fits “A room can host many meetings; a meeting uses exactly one room”?  

<v-click>

**Answer:** one-to-many (1 : n) relationship

</v-click>


---

### Participation Constraints

<br>

| Type                   | Notation                    | Meaning                                                                          |
| ---------------------- | --------------------------- | -------------------------------------------------------------------------------- |
| **Total (Mandatory)**  | Minimum cardinality = **1** | Every entity must participate in the relationship (e.g., `NOT NULL` foreign key) |
| **Partial (Optional)** | Minimum cardinality = **0** | Participation is optional; entity may or may not be involved (e.g., nullable FK) |

---

<br><br>


Explain the purpose of a **foreign key constraint** in SQL.

<br>


<v-click>

A foreign key constraint ensures that a column or set of columns in one table refers to a primary key in another table, maintaining **referential integrity** between the tables.

</v-click>





---

![](/crows_foot.png){width=600px lazy}



---

<br>

What does this tell us? What kind of relationship is this?

<br>

![](/image2.png){width=200px lazy}


<v-click>

Many-to-one (N:1) relationship with optional participation on both sides.

</v-click>

---


### Normalization Goals

<br>

Avoid anomalies:

<v-clicks>

- **Insert** Cannot add course without a professor  
- **Update** Change dept name in one row only  
- **Delete** Removing last prof deletes dept  

</v-clicks>

---

### Functional Dependency (FD)

<br>

`course_id  →  course_title, credits`  


<br>

<v-click>

> **Key Takeaway** “Every non‑key column depends on the *whole* key.”

</v-click>

---

### Exercise – Spot the Violation

<br>

```plain
in_dept(id, name, dept_name, dept_head)
```

<br>


<v-click>

**Answer:**

`id → name, dept_name, dept_head` — makes sense, since `id` uniquely identifies an instructor

But here's the problem: `dept_name → dept_head`

- Every department has one head ⇒ so `dept_name` determines `dept_head`

This means `dept_head` does not depend directly on the primary key, but on a non-key attribute (`dept_name`).

</v-click>

---


## Transitive dependency example:

<br>

```sql
student(student_id, name, advisor_name, advisor_office)
```
<br>


<v-click>

If `advisor_office` depends on `advisor_name`, and `advisor_name` depends on `student_id`, that’s a transitive dependency.


</v-click> 




---

## Normalization

A good reason to normalize a schema is:

(Choose one)

- improved query performance
- reduce redundancy
- reduce number of tables

<br>

<v-click>

answer is: "reduce redundancy"

</v-click>


---

<br><br>

Does this violate 1NF?

```sql
orders(order_id, customer_name, item_list)
```

<br>

<v-click>

Yes, `item_list` would likely contain a comma-separated list like "pen, notebook, eraser".


</v-click>

---

<br><br>

Explain the difference between Second Normal Form (2NF) and Third Normal Form (3NF).

<br>


<v-click>

2NF removes **partial dependencies**: every non-key attribute must depend on the whole primary key (relevant in composite keys).

</v-click>

<v-click>

3NF removes **transitive dependencies**: non-key attributes must only depend on the key, not on other non-key attributes.

</v-click> 



---

### Translating ER → Relational 

<br>


1. Entity set → table with PK  
2. 1 : n relationship → FK on “n” side  
3. m : n relationship → **junction table**  
4. Multi‑valued attribute → separate table  
5. Composite attribute → separate columns

---

### Design Section – Key Takeaways

<br>  

1.  Entities & attributes → tables & columns  
2.  Cardinality drives FK placement  
4.  FD ⇒ Normal forms ⇒ no anomalies  

---
layout: section
---

# Part 2  
## Relational Model & SQL
---

### Relational Algebra Primer

| Operator | Symbol | SQL |
|----------|--------|-----|
| Projection | 𝜋 | `SELECT col…` |
| Selection | 𝜎 | `WHERE …` |

<br>

> **Important** Relational Algebra treats relations as **sets** → duplicates eliminated.

---

<br><br>  

A relational database contains:

(Choose one)


- at most one table
- one or more tables

<br>

<v-click>

Answer is "one or more tables"

</v-click>



---


### DDL – Creating Tables

<br>

```sql
CREATE TABLE course (
  id      SERIAL PRIMARY KEY,
  title   TEXT NOT NULL,
  credits INT  CHECK (credits BETWEEN 1 AND 6),
  dept_id INT  REFERENCES department(id)
           ON DELETE CASCADE
);
```
<br>

> What does `ON DELETE CASCADE` do?

---

### Basic SELECT Template

<br>

```sql
SELECT   cols           -- 𝜋
FROM     tables         -- source
WHERE    predicate;     -- 𝜎
```

---

### Filtering 

<br>

| Want… | Syntax |
|-------|--------|
| Pattern | `LIKE '%help%'` |
| Range | `BETWEEN 10 AND 20` |
| NULL test | `IS NULL` |
| Remove dupes | `DISTINCT` |

---

### Aggregation & Grouping

<br>

```sql
SELECT dept_id, AVG(credits)
FROM   course
GROUP  BY dept_id
HAVING AVG(credits) > 3;
```

<v-click>

**WHERE vs. HAVING** – row filter vs. group filter


</v-click>

---

<br>

`WHERE`

  - Filters rows before any grouping or aggregation happens.
  - Can only be used with individual row values.
  - Works in the `SELECT ... FROM ... WHERE ...` clause.


<br>

<v-click>


`HAVING`
- Filters groups after GROUP BY and aggregation.
- Used to filter based on aggregate functions like COUNT(), AVG(), SUM().
- Must be used with GROUP BY.


</v-click>

---

## Examples

<br>

```sql
SELECT * FROM Employees
WHERE salary > 50000;
```

<br><br>

<v-click>

```sql
SELECT dept, AVG(salary) AS avg_salary
FROM Employees
GROUP BY dept
HAVING AVG(salary) > 60000;
```

</v-click>


---

### Joins Recap

```sql
SELECT s.name, c.title
FROM   student s
JOIN   takes   t ON t.sid = s.id
JOIN   course  c ON c.id = t.cid;
```

<br>

> Which join keeps *all left rows even if no match?* (answer next click)

<v-click>

Answer: **LEFT OUTER JOIN**

</v-click>

---

<br>

```sql
customers(customer_id, name)
orders(order_id, customer_id, total_amount)
```

Write a SQL query to list all customers and their orders, **even if the customer has no orders.**


<br>

<v-click>
Sample answer:

```sql
SELECT c.customer_id, c.name, o.order_id, o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

</v-click>

---


<br><br>

Explain the concept of a **correlated subquery**.

<v-click>

- A correlated subquery is a subquery that references a column from the outer query. 
- It is evaluated once for each row processed by the outer query.


</v-click>

---

### SQL Section – Key Takeaways

-  Basic SELECT‑FROM‑WHERE template  
- `GROUP BY` + `HAVING` for aggregates  
- Join types & when to use Outer joins  


---
layout: section
---

# Part 3  
## Implementation Details
---

### Storage 101

<br>

<v-clicks>

- **Sequential I/O** – contiguous blocks → cheap  
- **Random I/O** – seeks → costly on HDD, moderate on SSD  

</v-clicks>

---

### ACID & Concurrency

| Property | Guarantee |
|----------|-----------|
| Atomicity | all-or-nothing |
| Consistency | rules preserved |
| Isolation | concurrent ≈ serial |
| Durability | survives crash |

> PostgreSQL uses **WAL** + **MVCC**.

---

<br><br>

Briefly explain one potential anomaly that can occur in concurrent transactions without proper isolation.


<v-click>

**Answer:**

A potential anomaly is a "dirty read," where a transaction reads data that has been written by another transaction but has not yet been committed. If the second transaction is later rolled back, the first transaction has read invalid data. Another is a "lost update," where two transactions try to update the same data, and one transaction's changes are overwritten by the other.


</v-click>


---

### Locks & Deadlocks

<v-clicks>

- S (Shared) vs. X (Exclusive)  
- **Strict 2PL** – release after commit  
- Deadlock detection via waits‑for graph  

</v-clicks>

---

## Indexes

<br>

What is the main purpose of an **index** in a database?

<br>


<v-click>

An index is a data structure that improves the speed of data retrieval by providing a quick lookup mechanism to locate rows without scanning the entire table.

</v-click> 

---

Given this query, what type of index would help most?

```sql
SELECT * FROM Books WHERE price BETWEEN 10 AND 20;
```

<br>

<v-click>

A B-tree index would help most for this query.

The query performs a range scan on the price column. B-tree indexes are ideal for:

- Equality conditions (price = 15)
- Range conditions (`BETWEEN`, <, >, <=, >=)
- `ORDER BY` (if the sort matches the index order)

</v-click>

<br>

<v-click>

Hash indexes, in contrast:
- Are optimized only for equality lookups
- Cannot efficiently support range queries because hashing destroys order

</v-click>


---

Yes / No?

Is a database index useful for the query 

```sql
SELECT * FROM instructor;
```

<v-click>

Answer: no  (because for this query we need to scan the entire table)

</v-click>

---
layout: section
---

# Part 4  
## Beyond Relational
---

### NoSQL Motivation

<v-clicks>

- Massive scale / flexible schema  
- High availability under partitions  
- Schema‑on‑read vs. schema‑on‑write  

</v-clicks>

---


<br><br>

What is sharding in the context of distributed databases like MongoDB?


<v-click>

- Sharding is a horizontal partitioning technique that distributes data across multiple database instances or machines. 
- It improves scalability and load distribution by having each shard hold a portion of the overall dataset.

</v-click>


---

### Redis

<v-clicks>

- In‑memory ➜ µs latency  
- Data structures: String, Hash, List, Set, ZSet  
- Persistence: RDB snapshot or AOF log  
- Use cases: cache, session, Pub/Sub  

</v-clicks>

---

### ORMs & The N + 1

> **Problem** one parent row triggers N child queries.  
> **Fix** eager loading (`joinedload`, `selectinload`) *or* explicit joins.

---

### Beyond Section – Key Takeaways

- CAP ⇒ can’t have CA P at once  
- Mongo vs. JSONB = doc store vs. extensible column  
- Materialized view trades freshness for speed  
- Redis = RAM speed, watch memory footprint  
- ORMs safe, but watch N+1
