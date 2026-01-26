# SQL Mastery Notes – Aggregations, Joins, Subqueries & Window Functions

> **Context**: These notes summarize *all concepts learned and clarified* during the conversation.
> The goal is **revision-ready clarity**, **interview safety**, and **zero logical mistakes** in future SQL problems.

---

## 1. Conditional Aggregation (CORE CONCEPT)

### Definition

**Conditional aggregation** means applying aggregate functions over a group while selectively counting/summing rows based on a condition — **without filtering rows out using WHERE**.

> WHERE decides *which rows exist*.
> Conditional aggregation decides *how much each row contributes*.

### Key Patterns

```sql
SUM(condition)
SUM(CASE WHEN condition THEN value ELSE 0 END)
AVG(condition)
```

### Why It Matters

* Enables correct **percentages, ratios, metrics**
* Prevents losing denominator rows

### Common Mistake

❌ Using `WHERE condition` instead of conditional aggregation → wrong percentages

---

## 2. COUNT vs SUM vs AVG with Conditions

### Critical Truth

* `COUNT(expression)` counts **non-NULL values**, not TRUE values
* `SUM(condition)` works because TRUE = 1, FALSE = 0

### Comparison Table

| Expression       | Meaning             |
| ---------------- | ------------------- |
| COUNT(*)         | counts rows         |
| COUNT(col)       | counts non-NULL col |
| COUNT(condition) | ❌ usually wrong     |
| SUM(condition)   | ✅ conditional count |
| AVG(condition)   | ✅ percentage        |

### Golden Rule

> **COUNT checks NULLs, not truth.**

---

## 3. Self Join – Concept & Intuition

### When Self Join Is Needed

* Comparing rows **within the same table**
* Parent–child relationships (manager–employee)
* Start–end event pairs

### Example (LeetCode 570 – Managers with ≥ 5 Direct Reports)

```sql
SELECT m.name
FROM Employee m
JOIN Employee e ON m.id = e.managerId
GROUP BY m.id, m.name
HAVING COUNT(e.id) >= 5;
```

### Common Mistake

❌ Grouping by `m.managerId` instead of `m.id`

### Mental Model

> Self join = *two logical roles played by the same table*

---

## 4. ON vs WHERE in JOINs

### Execution Order (Mental Model)

1. FROM / JOIN → build row combinations
2. ON → decide matching
3. WHERE → remove rows

### Key Rule

* **ON** controls matching logic
* **WHERE** filters final rows

### LEFT JOIN Trap

Putting conditions in WHERE can turn LEFT JOIN into INNER JOIN.

---

## 5. Correlated vs Non-Correlated Subqueries

### Non-Correlated Subquery

* Independent of outer query
* Runs once

```sql
SELECT COUNT(*) FROM Users
```

### Correlated Subquery

* Depends on outer row
* Runs per row

```sql
WHERE d.order_date = (
  SELECT MIN(d2.order_date)
  FROM Delivery d2
  WHERE d2.customer_id = d.customer_id
)
```

### Mental Model

> **Correlated subquery = GROUP BY executed one group at a time**

---

## 6. Identifying “First Row per Group”

### Three Valid Approaches

1. Correlated subquery
2. JOIN with derived table
3. Window functions (preferred)

### Unsafe Pattern (Avoid)

```sql
WHERE order_date IN (SELECT MIN(order_date) ...)
```

❌ Loses row-to-group correlation

---

## 7. Window Functions – ROW_NUMBER

### Use Case

* First / last row per group
* Ranking
* De-duplication

### Example (LeetCode 1174 – Immediate Food Delivery II)

```sql
SELECT ROUND(AVG(order_date = customer_pref_delivery_date) * 100, 2)
FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) rn
  FROM Delivery
) t
WHERE rn = 1;
```

### Mental Model

> `ROW_NUMBER()` turns *row identification* into a simple filter

---

## 8. Percentages – Population Alignment (CRITICAL)

### Golden Rule

> **Numerator and denominator must refer to the SAME population**

### Common Bug

* Counting subset in numerator
* Using total table rows as denominator

### Fix

Always compute percentage **after reducing dataset to correct population**.

---

## 9. NULL Behavior in SQL

### Fundamental Rules

* NULL ≠ 0
* Any arithmetic with NULL → NULL

### Aggregates

| Function   | Behavior                                |
| ---------- | --------------------------------------- |
| SUM        | ignores NULLs, returns NULL if all NULL |
| COUNT(col) | ignores NULL                            |
| COUNT(*)   | counts rows                             |

### Safe Pattern

```sql
SUM(CASE WHEN condition THEN COALESCE(amount,0) ELSE 0 END)
```

---

## 10. Date-Based Self Joins (Weather, Activity Problems)

### Correct Date Join

```sql
curr.recordDate = DATE_ADD(prev.recordDate, INTERVAL 1 DAY)
```

### Why

* Dates are not numbers
* Avoid `date - 1`

---

## 11. Range Joins (Prices & UnitsSold)

### Pattern

```sql
purchase_date BETWEEN start_date AND end_date
```

### Use Case

* Price validity
* Slowly changing dimensions

### Key Insight

> Average selling price = **weighted average**, not average of prices

---

## 12. Window Functions vs Self Join vs Subquery

| Task                | Best Tool        |
| ------------------- | ---------------- |
| Aggregation         | GROUP BY         |
| Row comparison      | Window functions |
| Existence check     | EXISTS           |
| First row per group | ROW_NUMBER       |

---

## 13. Interview-Level Mental Checklist

Before finalizing any SQL:

1. What is my **population**?
2. Do numerator & denominator match?
3. Am I filtering rows incorrectly with WHERE?
4. Should this be conditional aggregation?
5. Can window functions simplify this?

---

## 14. LeetCode Problems Covered

* 570 – Managers with at Least 5 Direct Reports
* 1174 – Immediate Food Delivery II
* 1211 – Queries Quality and Percentage
* 177 – Nth Highest Salary (implicit concepts)
* 180 / 181 – Consecutive / Self Join logic
* 1965 – Employees With Missing Information
* Prices & UnitsSold – Average Selling Price
* Weather – Rising Temperature
* Activity – Average Processing Time

---

## Final Takeaway

> **SQL mastery is not syntax — it is population control, granularity control, and correct aggregation logic.**

These notes represent **interview-ready, production-safe SQL understanding**.

---

*End of revision document*
