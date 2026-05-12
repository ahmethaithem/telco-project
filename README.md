# 📡 i2i Systems - Telecom Database Management Project

## 📌 Project Overview
This project is a comprehensive database management system developed as part of a database assignment for **i2i Systems** (May 2026). The objective is to design, deploy, and manage a relational database for a telecommunications company using **Oracle Database 21c Express Edition (XE)**. 

The environment is containerized using **Docker**, ensuring a seamless, one-click setup. The database tracks mobile tariffs, customer details, and monthly usage statistics to provide actionable business insights.

---

## 🛠️ Technology Stack
*   **Database:** Oracle Database 21c (XE).
*   **Containerization:** Docker & Docker Compose.
*   **Administration Tool:** DBeaver.
*   **Query Language:** SQL (DDL & DML).

---

## 📂 Project Structure
```text
telco-project-main/
├── scripts/
│   └── 01_tables.sql       # Automated schema creation script
├── CUSTOMERS.csv           # Subscriber dataset
├── TARIFFS.csv             # Pricing plans dataset
├── MONTHLY_STATS.csv       # Usage and billing dataset
├── docker-compose.yml      # Container orchestration file
└── README.md               # Project documentation

🚀 How to Run the Project (A to Z)
Step 1: Start the Oracle Database Container
Launch the environment by running the following command in your terminal:

Bash
docker-compose up -d
Note: The database is mapped to port 1522 to avoid local port conflicts.

Step 2: Database Initialization
The script scripts/01_tables.sql executes automatically on startup to create the following schema:

TARIFFS: Contains data limits, voice minutes, and monthly fees.

CUSTOMERS: Contains subscriber profiles linked to specific tariffs.

MONTHLY_STATS: Tracks monthly data usage and payment statuses.

Step 3: Data Import Procedure
To ensure Relational Integrity and avoid ORA-02291 constraint errors, data was imported via DBeaver in the following mandatory order:

TARIFFS.csv (Parent table)

CUSTOMERS.csv (Child of Tariffs)

MONTHLY_STATS.csv (Child of Customers)

🔍 SQL Queries & Business Insights (Scenarios 1.1 - 1.7)
Below are the analytical queries derived to solve specific business scenarios:

1.1 Customer Distribution by City
Determines market penetration by counting subscribers in each city.

SQL
SELECT CITY, COUNT(CUSTOMER_ID) AS CUSTOMER_COUNT
FROM CUSTOMERS
GROUP BY CITY
ORDER BY CUSTOMER_COUNT DESC;
1.2 Total Monthly Income per Tariff
Calculates revenue generated per package to evaluate product performance.

SQL
SELECT t.NAME AS TARIFF_NAME, SUM(t.PRICE) AS TOTAL_MONTHLY_INCOME
FROM CUSTOMERS c
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
GROUP BY t.NAME
ORDER BY TOTAL_MONTHLY_INCOME DESC;
1.3 High Data Usage Alerts (75% Threshold)
Identifies users consuming more than 75% of their plan to target for potential upgrades.

SQL
SELECT c.NAME, m.DATA_USAGE, t.DATA_LIMIT
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
WHERE m.DATA_USAGE > (t.DATA_LIMIT * 0.75);
1.4 Early Adopter Analysis (Joined in 2023)
Focuses on subscribers who joined during the year 2023.

SQL
SELECT CITY, COUNT(*) AS EARLY_CUSTOMERS_COUNT
FROM CUSTOMERS
WHERE EXTRACT(YEAR FROM SIGNUP_DATE) = 2023
GROUP BY CITY;
1.5 Pending Payments Report
Identifies customers with an 'Unpaid' status for collection actions.

SQL
SELECT c.NAME, m.PAYMENT_STATUS
FROM CUSTOMERS c
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.PAYMENT_STATUS = 'Unpaid';
1.6 Integrity Audit (Missing Records)
Uses a LEFT JOIN to find customers who have no recorded usage data in the MONTHLY_STATS table.

SQL
SELECT c.CUSTOMER_ID, c.NAME
FROM CUSTOMERS c
LEFT JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
WHERE m.ID IS NULL;
1.7 Payment Status Breakdown by Package
Analyzes payment reliability across different tariff tiers.

SQL
SELECT t.NAME AS TARIFF_NAME, m.PAYMENT_STATUS, COUNT(c.CUSTOMER_ID) AS CUSTOMER_COUNT
FROM CUSTOMERS c
JOIN TARIFFS t ON c.TARIFF_ID = t.TARIFF_ID
JOIN MONTHLY_STATS m ON c.CUSTOMER_ID = m.ID
GROUP BY t.NAME, m.PAYMENT_STATUS
ORDER BY t.NAME, m.PAYMENT_STATUS;
🎯 Key Challenges & Solutions
Port Mapping: Resolved port conflicts by remapping Oracle to 1522 in docker-compose.yml.

Constraint Management: Solved ORA-02291 errors by establishing a strict data import order, ensuring parent records existed before importing dependent data.

Developed by AHMED MAHMOOD - May 2026
