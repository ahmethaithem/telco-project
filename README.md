# 📡 i2i Systems — Telecom Database Management Project

> **Database Assignment | May 2026**
> Developed by **Ahmed Haithem**

---

## 📌 Project Overview

A relational database system designed and managed for a telecommunications company using **Oracle Database 21c (XE)**. The environment is containerized via **Docker Compose** for easy reproducibility.

---

## 🛠️ Setup & Reproducibility

### 1. Docker XE Setup

The database runs in a Docker container using the provided `docker-compose.yml`.

```bash
docker-compose up -d
```

> **Note:** Mapped to port **1522** to avoid local conflicts.

---

### 2. Automated Initialization

The system automatically runs `scripts/01_tables.sql` upon container startup to create the full schema.

---

### 3. Data Import (DBeaver)

Data from the CSV files was imported using **DBeaver** in the following mandatory order to maintain referential integrity:

```
1. TARIFFS.csv        ← Parent table (import first)
2. CUSTOMERS.csv      ← Child of TARIFFS
3. MONTHLY_STATS.csv  ← Child of CUSTOMERS (import last)
```

---

## 🔍 Functional Requirements & SQL Queries

---

### 1. Tariff-Based Customer Queries

#### 1.1 — Customers in 'Kobiye Destek' Tariff

> Performs a join between customers and tariffs tables, filtering for the 'Kobiye Destek' package. Allows the business to identify the exact user base for this specific service plan.

```sql
SELECT c.*
FROM CUSTOMERS c
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
WHERE t.NAME = 'Kobiye Destek';
```

---

#### 1.2 — Newest Customer in 'Kobiye Destek'

> Sorts the filtered customer list by signup date in descending order, then isolates the single most recent addition. Useful for tracking recent marketing campaign successes.

```sql
SELECT *
FROM (
    SELECT c.*
    FROM CUSTOMERS c
    JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
    WHERE t.NAME = 'Kobiye Destek'
    ORDER BY c.SIGNUP_DATE DESC
)
WHERE ROWNUM = 1;
```

---

### 2. Tariff Distribution

#### 2.1 — Distribution of Tariffs Among Customers

> Groups subscriber data by tariff name and calculates the total count per plan. Helps management understand which packages are most popular in the market.

```sql
SELECT t.NAME, COUNT(c.CUSTOMER_ID) AS TOTAL_CUSTOMERS
FROM TARIFFS t
LEFT JOIN CUSTOMERS c ON t.TARIFF_ID = c.TARIFF_ID
GROUP BY t.NAME;
```

---

### 3. Customer Signup Analysis

#### 3.1 — Earliest Customers to Sign Up

> Sorts by signup date (not ID) to find the true first users in chronological order — avoids errors where IDs might not reflect the actual order of joining.

```sql
SELECT *
FROM CUSTOMERS
ORDER BY SIGNUP_DATE ASC;
```

---

#### 3.2 — Distribution of Earliest Customers by City

> Aggregates the earliest customers by city to show where initial company growth started. Provides geographic insight vital for historical trend analysis.

```sql
SELECT CITY, COUNT(*) AS CUSTOMER_COUNT
FROM CUSTOMERS
WHERE SIGNUP_DATE = (SELECT MIN(SIGNUP_DATE) FROM CUSTOMERS)
GROUP BY CITY;
```

---

### 4. Missing Monthly Records

#### 4.1 — Identify IDs of Missing Customers

> Uses a `LEFT JOIN` between customers and monthly usage tables. Filtering for `NULL` usage records exposes users affected by insertion errors — serves as an audit tool for data completeness.

```sql
SELECT c.CUSTOMER_ID
FROM CUSTOMERS c
LEFT JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.ID IS NULL;
```

---

#### 4.2 — Distribution of Missing Customers by City

> Counts customers with missing records grouped by city. Determines if the error was localized to a specific region and highlights areas requiring immediate data recovery.

```sql
SELECT c.CITY, COUNT(c.CUSTOMER_ID) AS MISSING_COUNT
FROM CUSTOMERS c
LEFT JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.ID IS NULL
GROUP BY c.CITY;
```

---

### 5. Usage Analysis

#### 5.1 — Customers Using at Least 75% of Data Limit

> Calculates the ratio between current usage and the allocated limit, filtering for values ≥ 0.75. Enables proactive customer support to offer package upgrades to heavy users.

```sql
SELECT c.NAME, m.DATA_USAGE, t.DATA_LIMIT
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
WHERE m.DATA_USAGE >= (t.DATA_LIMIT * 0.75);
```

---

#### 5.2 — Customers Exhausting All Limits (Data, Minutes, SMS)

> Checks three usage conditions simultaneously to find users who have reached or surpassed all caps. Identifying these "power users" is essential for capacity planning and revenue management.

```sql
SELECT c.NAME
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
WHERE m.DATA_USAGE   >= t.DATA_LIMIT
  AND m.MINUTE_USAGE >= t.MINUTE_LIMIT
  AND m.SMS_USAGE    >= t.SMS_LIMIT;
```

---

### 6. Payment Analysis

#### 6.1 — Customers with Unpaid Fees

> Filters directly on the `PAYMENT_STATUS` column for the value `'Unpaid'`, generating a collection list for the billing department.

```sql
SELECT c.NAME, m.PAYMENT_STATUS
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.PAYMENT_STATUS = 'Unpaid';
```

---

#### 6.2 — Distribution of Payment Statuses Across Tariffs

> Groups records by both tariff name and payment status. This multi-level aggregation reveals the financial health of each plan and identifies tariffs with higher rates of payment defaults.

```sql
SELECT t.NAME, m.PAYMENT_STATUS, COUNT(*) AS COUNT
FROM TARIFFS t
JOIN CUSTOMERS c ON t.TARIFF_ID = c.TARIFF_ID
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
GROUP BY t.NAME, m.PAYMENT_STATUS;
```

---

## 👨‍💻 Author

**Ahmed Haithem** — May 2026
