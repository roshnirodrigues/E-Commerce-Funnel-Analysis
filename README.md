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
```

## Calculating Conversion % by Device  
To determine if the conversion issues were device-specific, I utilized the FILTER clause.

```sql
SELECT COUNT(DISTINCT fs.session_id) FILTER (WHERE fs.event_step = 5) AS sessions_converted,
       COUNT(DISTINCT s.session_id) AS total_sessions,
       u.device,
       ROUND(
           COUNT(DISTINCT fs.session_id) FILTER (WHERE fs.event_step = 5) * 100.0
           / COUNT(DISTINCT s.session_id),
           2
       ) AS session_conversion_pct
FROM clean_sessions s
LEFT JOIN funnel_steps fs ON fs.session_id = s.session_id
LEFT JOIN clean_users u ON u.user_id = fs.user_id
GROUP BY u.device;

```
---

## 📊 Dashboard Pages
### Page 1: Conversion Overview

Provides a high-level view of conversion performance.

Includes:

Total sessions and converted sessions

Conversion rate, bounce rate, average order value

Converted sessions by device, city, traffic source, and user type

This page helps understand overall conversion performance and segment contributions.


### Page 2: Funnel & Drop-Off Analysis

Focuses on identifying where users exit the funnel.

Includes:

Funnel stage-wise session counts

Drop-off count at each stage

Cart abandonment rate

Drop-offs segmented by device, city, and traffic source

Navigation buttons improve usability across dashboard pages.

---

## 💡 Key Insights

- **Largest drop-off occurs between Product View and Add to Cart**

- **Second largest drop-off occurs at Checkout**

- **Drop-offs are similar across devices and traffic sources**

- Conversion issues appear funnel-wide rather than segment-specific

- Pricing perception and UX friction likely contribute to drop-offs

---

## 📈 Business Recommendations

### Optimise Product View → Add to Cart stage

- Improve pricing visibility

- Enhance product descriptions and imagery

- Validate add-to-cart functionality

### Reduce checkout friction

- Simplify checkout process

- Improve payment trust signals

- Remove unexpected costs

### Focus on funnel-wide improvements

- Segment optimisation alone will not solve the issue

---

## 📉 Business Impact: Prescriptive Scenario Analysis

Scenario analysis estimates potential improvements if recommendations are implemented.

### Scenario 1: Product View → Add to Cart

Current ATC conversion rate: 32%

Drop-off Reduction	Recovered Users	Potential ATC Conversion
5%	619	34%
10%	1,238	36%
15%	1,857	38%

Reducing drop-off by 10% improves ATC conversion by ~4%.

### Scenario 2: Cart Abandonment Recovery

Reduction	Recovered Orders	Recoverable Revenue
1%	59	₹1.85L
2%	117	₹3.70L
5%	293	₹9.27L

Even small improvements recover meaningful revenue.

---

## 🛠️ Tools Used

- PostgreSQL – joins, CTEs, window functions, views

- Excel – What-If Data Tables for scenarios

- Power BI – KPIs, funnel visuals, navigation
  
---

## ⚠️ Limitations

- Dataset is synthetic

- Scenario analysis is assumption-based

- Pricing and UX issues inferred indirectly


---
