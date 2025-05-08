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
- **Practice & Q‑A**  

</v-clicks>

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

![](/crows_foot.png){width=600px lazy}



---

<br>

What does this tell us? What kind of relationship is this?

<br>

![](/image2.png){width=200px lazy}


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

> **Key Takeaway** Know effects of `ON DELETE CASCADE`.

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
| Pattern | `LIKE 'Sm%th'` |
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



### Subqueries & CTEs – Quick Rules

<br>

1. `EXISTS` stops at first match – fast for big sets  
2. Scalar subquery must return **one row, one col**  
3. `WITH …` CTE improves readability; can be recursive

---

### SQL Section – Key Takeaways

-  Basic SELECT‑FROM‑WHERE template  
- `GROUP BY` + `HAVING` for aggregates  
- Join types & when to use Outer joins  
- Subquery vs. CTE choice affects clarity/perf  
- `EXPLAIN` is your friend

---

### Typical Exam Items (SQL)

- Write query: “Courses with zero students enrolled.”  
- Convert RA expression to SQL.  
- Explain difference between `IN` and `EXISTS`.

---

layout: section
# Part 3  
## Implementation Details
---

### Storage 101

<v-clicks>

- **Sequential I/O** – contiguous blocks → cheap  
- **Random I/O** – seeks → costly on HDD, moderate on SSD  
- **Exam‑critical 📝** Optimizer may still prefer Seq Scan for small tables (no seek).  

</v-clicks>

---

### Index Zoo

| Index | Good For | Notes |
|-------|----------|-------|
| **B‑Tree** | equality & range | default |
| **Hash** | equality only | rare in PG |
| **GIN** | arrays / JSONB | inverted |
| **GiST** | geo / full‑text | extensible |

---

### Reading an `EXPLAIN ANALYZE`

![plan placeholder](https://dummyimage.com/600x150/eeeeee/000000&text=plan+output+here "EXPLAIN plan") <!-- alt text for TTS -->

> **Tip** Look at *rows vs. rows=* estimate, buffers hit, actual time.

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

### Locks & Deadlocks

<v-clicks>

- S (Shared) vs. X (Exclusive)  
- **Strict 2PL** – release after commit  
- Deadlock detection via waits‑for graph  

</v-clicks>

---

### Implementation Section – Cheat Sheet

> 1️⃣ Sequential vs. Random I/O costs  
> 2️⃣ B‑Tree default index ⇢ equality & range  
> 3️⃣ Use `EXPLAIN ANALYZE` for timing  
> 4️⃣ ACID ⇢ WAL assures D & A  
> 5️⃣ Locks held to commit under Strict 2PL

---

### Typical Exam Items (Implementation)

- Interpret given `EXPLAIN ANALYZE` fragment.  
- Choose index type for JSON search.  
- Identify isolation level that allows “dirty read”.

---

layout: section
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

### CAP Snapshot

![cap](https://dummyimage.com/400x200/eeeeee/000000&text=CAP "CAP triangle")

> **Trade‑off example** MongoDB w/ majority write concern leans **CP**.

---

### MongoDB vs. JSONB Cheat Sheet

| Feature | MongoDB | PG JSONB |
|---------|---------|----------|
| Joins | `$lookup` | native |
| Txn scope | multi‑doc | full |
| Index path | multikey | GIN |
| Best for | flexible docs | hybrid workloads |

---

### Views & Materialized Views

<v-clicks>

- View = virtual, always fresh  
- Materialized = stored result  
- `REFRESH … CONCURRENTLY` for non‑blocking reads  

</v-clicks>

---

### Redis – 60‑Second Tour

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

> 1️⃣ CAP ⇒ can’t have CA P at once  
> 2️⃣ Mongo vs. JSONB = doc store vs. extensible column  
> 3️⃣ Materialized view trades freshness for speed  
> 4️⃣ Redis = RAM speed, watch memory footprint  
> 5️⃣ ORMs safe, but watch N+1

---

### Typical Exam Items (Beyond)

- State CAP theorem; classify Redis.  
- Compare `$lookup` to SQL `JOIN`.  
- Explain benefit of materialized view + example.

---

layout: section
# Practice & Q ± A
---

### Hands‑On Tasks

1. **ER Drawing** – dorm‑room schema  
2. **Normalize** `in_dept` to BCNF  
3. **Write** CTE calculating avg credits per dept (>3)  
4. **Interpret** provided `EXPLAIN ANALYZE` plan

(To be solved live / in breakout groups.)

---

layout: center
# Still Unclear?

*Parking‑lot slide* – jot your lingering questions here so we can tackle them.

---
