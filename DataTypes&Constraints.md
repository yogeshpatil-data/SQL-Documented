# 🚀 DATA ENGINEERING MASTER GUIDE

## Constraints, Data Types, Casting, Dates & Partitioning (Production + Interview Level)

---

# 0. PURPOSE OF THIS GUIDE

This is a **complete, production-grade reference** to:

* Understand **how data behaves across systems**
* Prevent **silent data corruption**
* Design **robust data pipelines**
* Answer **interview questions with depth**

---

# 1. SYSTEM BEHAVIOR OVERVIEW (FOUNDATION)

| System                | Nature            | Priority                  |
| --------------------- | ----------------- | ------------------------- |
| OLTP (Postgres/MySQL) | Transactional     | Data correctness          |
| Snowflake             | Analytical        | Performance & flexibility |
| Delta Lake            | Data lake + ACID  | Scalability + control     |
| PySpark               | Processing engine | Transformation            |

---

# 2. CONSTRAINTS – COMPLETE TRUTH

---

## 2.1 What Constraints Really Do

Constraints are **guarantees enforced by system OR expected by pipeline**

👉 Critical distinction:

* OLTP → **enforced guarantees**
* Warehouse/Data Lake → **assumed guarantees**

---

## 2.2 OLTP Constraints (STRICT ENFORCEMENT)

Examples: PostgreSQL, MySQL

### Behavior

* Enforced during INSERT/UPDATE
* Violations → transaction rollback

---

### Example

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY,
  user_id INT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

### Guarantees

* No duplicates
* Referential integrity
* Clean data always

---

### Internal Mechanism

* Indexes enforce uniqueness
* Constraint checks in execution plan
* Transaction isolation ensures consistency

---

---

## 2.3 Snowflake Constraints (LOGICAL ONLY)

Example: Snowflake

---

### Behavior

* Metadata only
* Not validated on write

---

### Reality

```sql
PRIMARY KEY (id)
```

👉 Stored but ignored

---

### Why?

* Distributed architecture
* Columnar storage
* Massive ingestion

---

### Practical Impact

* Duplicate records possible
* Broken relationships possible

---

👉 Responsibility shifts to **data pipeline**

---

---

## 2.4 Delta Lake Constraints (SELECTIVE ENFORCEMENT)

Example: Delta Lake

---

### Supported

* NOT NULL ✅
* CHECK ✅

---

### Not Supported

* PRIMARY KEY ❌
* FOREIGN KEY ❌
* UNIQUE ❌

---

### Internal Behavior

* Constraints validated during write
* Stored in transaction log

---

### Example

```sql
ALTER TABLE users ADD CONSTRAINT valid_age CHECK (age > 18);
```

---

---

## 2.5 Real Production Strategy

---

### How to Simulate PRIMARY KEY

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number

df = df.withColumn(
    "rn",
    row_number().over(Window.partitionBy("id").orderBy("updated_at"))
).filter("rn = 1")
```

---

### Use MERGE for Upserts

```sql
MERGE INTO target t
USING source s
ON t.id = s.id
```

---

---

## 🔥 Final Constraint Truth

> In modern data engineering, constraints are NOT enforced by systems—you enforce them in pipelines.

---

---

# 3. DATA TYPES – DEEP UNDERSTANDING

---

# 3.1 NUMERIC TYPES

---

## Integer

* Exact values
* Fast
* Used for IDs, counts

---

## FLOAT / DOUBLE (Approximate Types)

---

### Internal Representation

```text
Value = Sign × Mantissa × 2^Exponent
```

---

### Problem

```text
0.1 → infinite binary → approximation
```

---

### Result

```text
0.1 + 0.2 = 0.30000000000000004
```

---

### Differences

| Type   | Precision  |
| ------ | ---------- |
| FLOAT  | ~7 digits  |
| DOUBLE | ~16 digits |

---

### When to Use

* Scientific calculations
* ML features

---

---

## DECIMAL (Exact Type)

---

### Definition

```sql
DECIMAL(precision, scale)
```

---

### Behavior

* Exact storage
* No rounding error

---

### Trade-off

* Slower than DOUBLE
* More storage

---

---

# 3.2 STRING TYPES

---

| Type    | Behavior     |
| ------- | ------------ |
| CHAR    | Fixed length |
| VARCHAR | Variable     |
| TEXT    | Large        |

---

### Important Insight

In warehouses:

* VARCHAR length is often **not enforced**

---

---

# 3.3 DATE & TIMESTAMP

---

## Internal Representation

| Type      | Storage                  |
| --------- | ------------------------ |
| DATE      | Days since epoch         |
| TIMESTAMP | Microseconds since epoch |

---

---

## Important Behavior

* Always displayed as:

```text
yyyy-MM-dd
```

---

---

# 3.4 COMPLEX TYPES (SPARK)

---

## Struct

Nested object

## Array

List of values

## Map

Key-value pairs

---

---

# 4. TYPE CASTING – INTERNAL BEHAVIOR

---

# 4.1 How Casting Works Internally

In Apache Spark:

* Logical plan → Catalyst optimizer
* Physical execution → Cast expression
* Invalid → NULL

---

---

# 4.2 Critical Rule

> Casting NEVER throws error → returns NULL

---

---

# 4.3 Examples

---

## String → Integer

```python
"123" → 123
"abc" → NULL
```

---

## String → Date

```python
to_date(col, "yyyy-MM-dd")
```

Wrong format → NULL

---

---

## Float → Decimal

```text
123.456 → 123.46
```

---

---

# 4.4 Hidden Dangers

---

## Silent Data Loss

NULL values introduced silently

---

## Join Failures

```text
string vs int mismatch
```

---

## Aggregation Errors

NULL ignored in SUM

---

---

# 4.5 Validation Pattern (MANDATORY)

```python
df.filter(col("casted_col").isNull())
```

---

---

# 5. DATE HANDLING – FULL GUIDE

---

# 5.1 Parsing vs Formatting

| Function    | Purpose       |
| ----------- | ------------- |
| to_date     | STRING → DATE |
| date_format | DATE → STRING |

---

---

# 5.2 Pattern Cheat Sheet

---

## SAFE

```text
yyyy-MM-dd HH:mm:ss
```

---

## DANGEROUS

| Pattern | Issue       |
| ------- | ----------- |
| YYYY    | Week year   |
| mm      | Minutes     |
| DD      | Day of year |

---

---

# 5.3 Multi-Format Parsing

```python
coalesce(
  to_date(col, "yyyy-MM-dd"),
  to_date(col, "MM-dd-yyyy")
)
```

---

---

# 5.4 Timezone Issues

* Spark uses session timezone
* Data shifts possible

---

👉 Always standardize to UTC

---

---

# 6. PARTITIONING – PRODUCTION DESIGN

---

# 6.1 Goal

* Reduce scan
* Improve performance

---

---

# 6.2 BAD

```python
partitionBy("date")
```

---

Problems:

* Too many partitions
* Metadata explosion

---

---

# 6.3 GOOD

```python
partitionBy("year", "month", "day")
```

---

---

# 6.4 ADVANCED STRATEGY

---

## Large Data

* Partition by year/month
* Use Z-order for fine filtering

---

---

## Streaming Pipelines

* Partition by ingestion date
* Not event date

---

---

# 6.5 Partition Pruning

---

Query:

```sql
WHERE date = '2025-01-10'
```

---

Spark reads only:

```text
/year=2025/month=01/day=10
```

---

---

# 7. DATA ENGINEERING ARCHITECTURE (IMPORTANT)

---

## Bronze Layer

* Raw data
* STRING / VARIANT

---

## Silver Layer

* Cleaned
* Typed
* Validated

---

## Gold Layer

* Aggregated
* Business-ready

---

---

# 8. COMMON PRODUCTION FAILURES

---

## 1. Silent NULL Explosion

* Casting failures

---

## 2. Wrong Date Patterns

* mm vs MM
* YYYY vs yyyy

---

## 3. Precision Loss

* DOUBLE used for money

---

## 4. Schema Drift

* Type changes across days

---

## 5. Partition Explosion

* Over-partitioning

---

---

# 9. FINAL MASTER MENTAL MODEL

---

## Constraints

* OLTP → enforced
* Warehouse → ignored
* Delta → partial

---

## Data Types

* FLOAT/DOUBLE → approximate
* DECIMAL → exact

---

## Casting

* Unsafe by default
* Must validate

---

## Dates

* Always parse explicitly
* Never rely on defaults

---

## Partitioning

* Balance size vs performance

---

---

# 🔥 FINAL GOLDEN RULES

---

1. **Never trust incoming data types**
2. **Always validate after casting**
3. **Avoid FLOAT for critical data**
4. **Always use yyyy-MM-dd**
5. **Never rely on DB constraints in warehouses**
6. **Partition smartly, not blindly**

---

---

# 🎯 FINAL INTERVIEW SUMMARY

> In modern data engineering systems like Snowflake and Delta Lake, constraints are not strictly enforced, shifting responsibility to the pipeline. Data types must be carefully handled, especially approximate types like float/double, and casting must be validated due to Spark's silent null behavior. Proper date handling and partitioning strategies are essential to ensure both correctness and performance.

---

---

This is your **central reference sheet**.
If you master this, you are operating at **top-tier Data Engineer level (3–5 YOE)**.

---
