# Amazon Sales SQL Practice Questions (Intermediate → Advanced)

---

## Q1. Which category brings in the most revenue,
-- and what is the % contribution of each to overall revenue?
---
```sql
SELECT 
    category,
    ROUND(SUM(total_amount), 2) AS total_revenue
FROM amazon_sales
GROUP BY category
ORDER BY total_revenue DESC;
```
---
