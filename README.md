# 📡 i2i Systems — Telecom Database Management Project

> **Database Assignment | May 2026**
> Designed, deployed, and managed by **Ahmed Mahmood**

---

## 📌 Project Overview

A comprehensive relational database system built for a telecommunications company using **Oracle Database 21c Express Edition (XE)**. The environment is containerized with Docker for seamless one-click setup.

The database tracks:
- 📋 Mobile tariffs & pricing plans
- 👥 Customer profiles & subscriptions
- 📊 Monthly usage statistics & billing

---

## 🛠️ Technology Stack

| Tool | Purpose |
|------|---------|
| **Oracle Database 21c XE** | Core relational database engine |
| **Docker & Docker Compose** | Containerization & environment setup |
| **DBeaver** | Database administration & data import |
| **SQL (DDL & DML)** | Schema creation & data queries |

---

## 📂 Project Structure

```
telco-project-main/
├── scripts/
│   └── 01_tables.sql       # Automated schema creation script
├── CUSTOMERS.csv           # Subscriber dataset
├── TARIFFS.csv             # Pricing plans dataset
├── MONTHLY_STATS.csv       # Usage and billing dataset
├── docker-compose.yml      # Container orchestration file
└── README.md               # Project documentation
```

---

## 🚀 How to Run the Project

### Step 1 — Start the Oracle Database Container

```bash
docker-compose up -d
```

> **Note:** The database is mapped to port **1522** to avoid local port conflicts.

---

### Step 2 — Database Initialization

The script `scripts/01_tables.sql` executes **automatically on startup** and creates the following tables:

| Table | Description |
|-------|-------------|
| `TARIFFS` | Data limits, voice minutes, and monthly fees |
| `CUSTOMERS` | Subscriber profiles linked to specific tariffs |
| `MONTHLY_STATS` | Monthly data usage and payment statuses |

---

### Step 3 — Data Import Order ⚠️

To maintain **Relational Integrity** and avoid `ORA-02291` constraint errors, import data in this **mandatory order** via DBeaver:

```
1. TARIFFS.csv        ← Parent table (import first)
2. CUSTOMERS.csv      ← Child of TARIFFS
3. MONTHLY_STATS.csv  ← Child of CUSTOMERS (import last)
```

---

## 🔍 SQL Queries & Business Insights

### 1.1 — Customer Distribution by City

Determines market penetration by counting subscribers per city.

```sql
SELECT CITY, COUNT(CUSTOMER_ID) AS CUSTOMER_COUNT
FROM CUSTOMERS
GROUP BY CITY
ORDER BY CUSTOMER_COUNT DESC;
```

---

### 1.2 — Total Monthly Income per Tariff

Calculates revenue generated per package to evaluate product performance.

```sql
SELECT t.NAME AS TARIFF_NAME, SUM(t.PRICE) AS TOTAL_MONTHLY_INCOME
FROM CUSTOMERS c
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
GROUP BY t.NAME
ORDER BY TOTAL_MONTHLY_INCOME DESC;
```

---

### 1.3 — High Data Usage Alerts (75% Threshold)

Identifies users consuming more than 75% of their data plan — targets for upgrade offers.

```sql
SELECT c.NAME, m.DATA_USAGE, t.DATA_LIMIT
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
WHERE m.DATA_USAGE > (t.DATA_LIMIT * 0.75);
```

---

### 1.4 — Early Adopter Analysis (Joined in 2023)

Focuses on subscribers who joined during the year 2023.

```sql
SELECT CITY, COUNT(*) AS EARLY_CUSTOMERS_COUNT
FROM CUSTOMERS
WHERE EXTRACT(YEAR FROM SIGNUP_DATE) = 2023
GROUP BY CITY;
```

---

### 1.5 — Pending Payments Report

Identifies customers with an `Unpaid` status for collection actions.

```sql
SELECT c.NAME, m.PAYMENT_STATUS
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.PAYMENT_STATUS = 'Unpaid';
```

---

### 1.6 — Integrity Audit (Missing Records)

Uses a `LEFT JOIN` to find customers with no recorded usage data in `MONTHLY_STATS`.

```sql
SELECT c.CUSTOMER_ID, c.NAME
FROM CUSTOMERS c
LEFT JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.ID IS NULL;
```

---

### 1.7 — Payment Status Breakdown by Package

Analyzes payment reliability across different tariff tiers.

```sql
SELECT t.NAME AS TARIFF_NAME, m.PAYMENT_STATUS, COUNT(c.CUSTOMER_ID) AS CUSTOMER_COUNT
FROM CUSTOMERS c
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
GROUP BY t.NAME, m.PAYMENT_STATUS
ORDER BY t.NAME, m.PAYMENT_STATUS;
```

---

## 🎯 Key Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| **Port Conflict** | Remapped Oracle from default 1521 to port **1522** in `docker-compose.yml` |
| **ORA-02291 Errors** | Established a strict import order ensuring parent records exist before dependent data |

---

## 👨‍💻 Author

**Ahmed Mahmood** — May 2026
