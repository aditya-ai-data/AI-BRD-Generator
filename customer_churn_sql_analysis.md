# SaaS Customer Churn Analysis (SQL)

## Business Question
Which customer behaviors are most associated with churn in a SaaS product?

---

## Assumed Table Schema

**customers**
- customer_id (INT, primary key)
- signup_date (DATE)
- country (VARCHAR)

**subscriptions**
- customer_id (INT)
- plan (VARCHAR)
- start_date (DATE)
- end_date (DATE)
- status (VARCHAR)

**usage_events**
- event_id (INT, primary key, auto-increment)
- customer_id (INT)
- event_date (DATE)
- event_type (VARCHAR)

---

## SQL Analysis

### 1) Churn rate by plan

```sql
SELECT
  plan,
  COUNT(*) AS total_customers,
  SUM(status = 'canceled') AS churned_customers,
  ROUND(SUM(status = 'canceled') / COUNT(*), 2) AS churn_rate
FROM subscriptions
GROUP BY plan;
````

**Output (sample run):**

* basic, 3, 2, 0.67
* pro, 2, 1, 0.50

**Interpretation:**
The Basic plan shows a higher churn rate (67%) compared to the Pro plan (50%), suggesting that lower-tier customers may be experiencing a value gap or are more price-sensitive. This indicates potential issues with onboarding, feature limitations, or perceived ROI for Basic users, and suggests that improving early value delivery or adjusting plan differentiation could reduce churn.

---

### 2) Average time to churn

```sql
SELECT
  ROUND(AVG(DATEDIFF(end_date, start_date)), 1) AS avg_days_before_churn
FROM subscriptions
WHERE status = 'canceled';
```

**Output (sample run):**

* avg_days_before_churn = 57.3

**Interpretation:**
Churned customers cancel after an average of 57.3 days, indicating that users typically engage with the product for nearly two months before leaving. This suggests that while initial onboarding is sufficient to get users started, the product may struggle to deliver sustained value over time, highlighting an opportunity to improve engagement, feature stickiness, or ongoing customer success beyond the first month.

---

### 3) Usage comparison: churned vs active customers

```sql
SELECT
  s.status,
  COUNT(u.event_id) / COUNT(DISTINCT s.customer_id) AS avg_events
FROM subscriptions s
LEFT JOIN usage_events u
  ON s.customer_id = u.customer_id
GROUP BY s.status;
```

**Output (sample run):**

* active, 0.5000
* canceled, 1.3333

**Interpretation:**
In this sample dataset, churned customers show a higher average number of usage events than active customers. This indicates that total usage volume alone is not a reliable indicator of retention, particularly in small datasets. Churned users may exhibit burst usage as they explore the product before deciding to cancel, highlighting the need to analyze usage patterns and engagement recency rather than raw event counts.

---

### 4) Activity in the last 30 days before churn

```sql
SELECT
  s.customer_id,
  COUNT(u.event_id) AS events_last_30_days
FROM subscriptions s
LEFT JOIN usage_events u
  ON s.customer_id = u.customer_id
 AND u.event_date BETWEEN DATE_SUB(s.end_date, INTERVAL 30 DAY) AND s.end_date
WHERE s.status = 'canceled'
GROUP BY s.customer_id;
```

**Output (sample run):**

* 1, 2
* 3, 1
* 4, 0

**Interpretation:**
Analysis of activity in the 30 days preceding churn shows that all churned customers exhibited low or declining engagement, with one customer having no recorded activity at all. This demonstrates that reduced recent usage is a stronger and more actionable churn signal than total historical usage, and suggests that monitoring engagement drop-offs could enable earlier retention interventions.

---

## Business Takeaways

* Churn rates are higher among lower-tier plan customers, indicating a potential value gap for Basic users.
* Customers typically churn after approximately two months, suggesting challenges in sustaining long-term value rather than initial onboarding.
* Total usage volume alone is not a reliable churn predictor; engagement recency provides a stronger signal.
* Monitoring declining activity in the final 30 days before churn can enable proactive retention strategies and targeted interventions.
