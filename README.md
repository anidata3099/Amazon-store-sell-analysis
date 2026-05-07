# Amazon Sales SQL Practice Questions (Intermediate → Advanced)

---

## Q1. Which category brings in the most revenue,
-- and what is the % contribution of each to overall revenue?
---
```sql
SELECT
    category,
    COUNT(order_id) AS total_orders,
    SUM(total_amount) AS category_revenue,
    SUM(total_amount) * 100.0
        / SUM(SUM(total_amount)) OVER () AS revenue_pct
FROM amazon
GROUP BY category
ORDER BY category_revenue DESC;
```
| category          | total_orders | category_revenue | revenue_pct      |
| ----------------- | ------------ | ---------------- | ---------------- |
| Electronics       | 31683        | 29260310.78      | 16.9566496959991 |
| Sports & Outdoors | 31564        | 28807925.2499998 | 16.6944876493473 |
| Clothing          | 30930        | 28714739.9699999 | 16.6404858254548 |
| Books             | 31490        | 28687489.6200001 | 16.6246939686111 |
| Toys & Games      | 31029        | 28562069.0299999 | 16.5520114525131 |
| Home & Kitchen    | 31210        | 28526970.3500001 | 16.5316714080746 |
