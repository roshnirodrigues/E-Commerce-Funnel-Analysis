# 🛒 E-Commerce Conversion & Funnel Analysis

## 🚀 Project Overview

This project analyses an e-commerce funnel to understand why high traffic does not translate into proportional conversions and sales. The focus was on identifying where users drop off during the purchase journey and whether these drop-offs are driven by specific segments or by broader funnel issues.

This is an end-to-end analytics project using:

- PostgreSQL for data cleaning, preparation, and analysis  
- Excel for What-If / scenario modelling  
- Power BI for dashboarding and visualisation  

The dataset used is synthetic, created to simulate realistic e-commerce user behaviour.

---

## 🎯 Business Problem

Despite strong traffic, the platform experiences low conversion rates, resulting in lower sales. The business needs to understand:

- Where users drop off in the funnel
- Whether drop-offs are driven by device, city, or traffic source
- Whether issues are segment-specific or funnel-wide
- Which stages should be prioritised for optimisation

---

## 🗂️ Data Description

The dataset consists of multiple related tables representing user behaviour across the e-commerce funnel:

- **users** – user-level attributes (user id, device, traffic source, city, user type)
- **sessions** – session-level information (session id, duration, pages viewed, timestamps)
- **funnel_events** – event-level funnel actions (event type, status)
- **orders** – successful purchases and order values

Together, these tables represent the funnel:

**Visit → Product View → Add to Cart → Checkout → Payment Success**

---

## 🧹 Data Cleaning & Preparation (PostgreSQL)

All data cleaning and transformation were performed in PostgreSQL.

Initial steps included understanding dataset structure and relationships.

Key preparation steps:

- Checked for null values, duplicates, and inconsistencies
- Removed duplicate records
- Replaced null values in device and traffic source with `"Unknown"`
- Fixed inconsistent categorical values

To support analysis:

- Funnel stages were segmented using `CASE` statements
- Derived fields were created for:
  - Converted sessions
  - Funnel step flags
  - Drop-off counts

Clean SQL views were created for reusable funnel metrics.

Final tables were created with constraints and imported into Power BI.

### SQL Techniques Used

- CASE statements
- Common Table Expressions (CTEs)
- Window functions
- GROUP BY aggregations
- JOIN operations
- Views for reusable datasets

---

## 🔍 Analysis Approach

Key funnel metrics calculated using SQL:

- Total sessions and converted sessions
- Conversion rate
- Bounce rate
- Cart abandonment rate
- Drop-off counts by funnel stage

Funnel performance was segmented by:

- Device
- Traffic source
- City
- User type (new vs returning)

Aggregated tables were then imported into Power BI for visualisation.

---

## 💻 SQL Code Snippets

### Drop-Off % Calculation Between Funnel Steps

```sql
WITH count_per_step AS (
    SELECT event_type, event_step,
           COUNT(DISTINCT session_id) AS sessions
    FROM funnel_steps
    GROUP BY event_step, event_type
)

SELECT event_step,
       sessions,
       (lag(sessions) OVER (ORDER BY event_step) - sessions) AS drop_off_count,
       ROUND(
           (lag(sessions) OVER (ORDER BY event_step) - sessions) * 100.0
           / lag(sessions) OVER (ORDER BY event_step),
           2
       ) AS drop_off_pct
FROM count_per_step
ORDER BY event_step;
