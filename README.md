# Amazon Sales SQL Practice Questions (Basic → Intermediate → Advanced)

---

## Q1. Top 5 Selling Products by Revenue + Contribution %
---
```sql
WITH product_rev AS (
    SELECT 
        product_name,
        category,
        SUM(total_amount) AS revenue,
        SUM(quantity) AS units_sold
    FROM amazon
    GROUP BY product_name, category
)
SELECT 
    product_name,
    category,
    revenue,
    ROUND((100.0 * revenue / SUM(revenue) OVER ())::numeric, 2) AS revenue_contribution_pct,
    RANK() OVER (ORDER BY revenue DESC) AS rank
FROM product_rev
ORDER BY revenue DESC
LIMIT 5;
```
| product_name    | category          | revenue   | revenue_contribution_pct | rank |
| --------------- | ----------------- | --------- | ------------------------ | ---- |
| Jeans           | Sports & Outdoors | 695515.49 | 0.40                     | 1    |
| Microphone      | Sports & Outdoors | 668127.71 | 0.39                     | 2    |
| Office Chair    | Clothing          | 665767.73 | 0.39                     | 3    |
| Smartphone Case | Books             | 665140.76 | 0.39                     | 4    |
| 4K Monitor      | Toys & Games      | 664534.23 | 0.39                     | 5    |



## Q2. Customer Segmentation by Total Spend (RFM - Monetary only)
---
```SQL
SELECT 
    customer_id,
    customer_name,
    SUM(total_amount) AS total_spend,
    COUNT(DISTINCT order_id) AS order_count,
    NTILE(5) OVER (ORDER BY SUM(total_amount) DESC) AS monetary_segment
FROM amazon
GROUP BY customer_id, customer_name
ORDER BY total_spend DESC;
```
| customer_id | customer_name | total_spend | order_count | monetary_segment |
| ----------- | ------------- | ----------- | ----------- | ---------------- |
| CUST010696  | Pooja Patel   | 10872.08    | 2           | 1                |
| CUST035973  | Neha Sharma   | 10248.86    | 2           | 1                |
| CUST018720  | Arjun Kapoor  | 9546.36     | 2           | 1                |
| CUST022824  | Pooja Joshi   | 9510.96     | 2           | 1                |
| CUST008882  | Sneha Reddy   | 9460.94     | 2           | 1                |

## Q3. Average Order Value (AOV) by Category and Payment Method
---
```sql
SELECT 
    category,
    payment_method,
    COUNT(DISTINCT order_id) AS total_orders,
    SUM(total_amount) AS total_revenue,
    ROUND(SUM(total_amount)::numeric / COUNT(DISTINCT order_id), 2) AS aov
FROM amazon
GROUP BY category, payment_method
ORDER BY total_revenue DESC limit 5;
```
| category          | payment_method | total_orders | total_revenue | aov     |
| ----------------- | -------------- | ------------ | ------------- | ------- |
| Electronics       | Credit Card    | 5900         | 10293391.62   | 1744.64 |
| Books             | Credit Card    | 5939         | 10099095.57   | 1700.47 |
| Sports & Outdoors | Credit Card    | 5915         | 10099007.43   | 1707.36 |
| Clothing          | Credit Card    | 5811         | 10089830.93   | 1736.33 |
| Home & Kitchen    | Credit Card    | 5765         | 9963418.48    | 1728.26 |

## Q4.Cancellation Rate by Category and State
---
``` SQL
SELECT 
    category,
    state,
    COUNT(*) AS total_orders,
    SUM(CASE WHEN order_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled,
    ROUND(100.0 * SUM(CASE WHEN order_status = 'Cancelled' THEN 1 ELSE 0 END) / COUNT(*), 2) AS cancel_rate_pct
FROM amazon
GROUP BY category, state
HAVING COUNT(*) >= 50
ORDER BY cancel_rate_pct DESC limit 5;
```
| category          | state | total_orders | cancelled | cancel_rate_pct |
| ----------------- | ----- | ------------ | --------- | --------------- |
| Sports & Outdoors | OH    | 1606         | 76        | 4.73            |
| Toys & Games      | IN    | 1465         | 64        | 4.37            |
| Sports & Outdoors | CO    | 1497         | 65        | 4.34            |
| Books             | NY    | 1520         | 66        | 4.34            |
| Clothing          | NY    | 1466         | 61        | 4.16            |

## Q5.Seller Performance - Revenue + Order Count
---
```SQL
SELECT 
    seller_id,
    COUNT(DISTINCT order_id) AS orders,
    SUM(total_amount) AS revenue,
    ROUND(SUM(total_amount)::numeric / COUNT(DISTINCT order_id), 2) AS avg_order_value,
    DENSE_RANK() OVER (ORDER BY SUM(total_amount) DESC) AS revenue_rank
FROM amazon
GROUP BY seller_id
ORDER BY revenue DESC limit 5;
```
| seller_id | orders | revenue   | avg_order_value | revenue_rank |
| --------- | ------ | --------- | --------------- | ------------ |
| SELL00440 | 61     | 137220.82 | 2249.52         | 1            |
| SELL00536 | 70     | 136446.87 | 1949.24         | 2            |
| SELL01164 | 60     | 133036.69 | 2217.28         | 3            |
| SELL00812 | 68     | 131583.37 | 1935.05         | 4            |
| SELL00806 | 67     | 130748.26 | 1951.47         | 5            |

## Q6.Discount Impact Analysis
---
```SQL
SELECT 
    category,
    ROUND(AVG(discount)::numeric, 2) AS avg_discount_pct,
    ROUND(AVG(CASE WHEN discount > 0 THEN total_amount ELSE NULL END)::numeric, 2) AS avg_revenue_with_discount,
    ROUND(AVG(CASE WHEN discount = 0 THEN total_amount ELSE NULL END)::numeric, 2) AS avg_revenue_without_discount
FROM amazon
GROUP BY category
ORDER BY avg_discount_pct DESC;
```
| category          | avg_discount_pct | avg_revenue_with_discount | avg_revenue_without_discount |
| ----------------- | ---------------- | ------------------------- | ---------------------------- |
| Home & Kitchen    | 0.08             | 864.98                    | 988.87                       |
| Books             | 0.07             | 867.21                    | 976.44                       |
| Clothing          | 0.07             | 884.73                    | 991.95                       |
| Electronics       | 0.07             | 875.42                    | 993.37                       |
| Sports & Outdoors | 0.07             | 867.46                    | 980.24                       |
| Toys & Games      | 0.07             | 868.45                    | 998.08    

## Q7.RFM Analysis (Recency, Frequency, Monetary)
---
```SQL
WITH rfm_base AS (
    SELECT 
        customer_id,
        MAX(order_date) AS last_order_date,
        COUNT(DISTINCT order_id) AS frequency,
        SUM(total_amount) AS monetary
    FROM amazon
    GROUP BY customer_id
),
rfm_score AS (
    SELECT 
        customer_id,
        NTILE(5) OVER (ORDER BY last_order_date DESC) AS recency_score,      -- 5 = most recent
        NTILE(5) OVER (ORDER BY frequency DESC) AS frequency_score,
        NTILE(5) OVER (ORDER BY monetary DESC) AS monetary_score
    FROM rfm_base
)
SELECT 
    customer_id,
    recency_score,
    frequency_score,
    monetary_score,
    (recency_score + frequency_score + monetary_score) AS rfm_total,
    CASE 
        WHEN recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4 THEN 'Champions'
        WHEN recency_score >= 4 AND frequency_score >= 3 THEN 'Loyal Customers'
        WHEN recency_score <= 2 AND frequency_score <= 2 THEN 'At Risk'
        ELSE 'Others'
    END AS segment
FROM rfm_score limit 10;
```
| customer_id | recency_score | frequency_score | monetary_score | rfm_total | segment |
| ----------- | ------------- | --------------- | -------------- | --------- | ------- |
| CUST025919  | 1             | 1               | 1              | 3         | At Risk |
| CUST017994  | 1             | 2               | 2              | 5         | At Risk |
| CUST020361  | 1             | 1               | 2              | 4         | At Risk |
| CUST008413  | 1             | 1               | 1              | 3         | At Risk |
| CUST027587  | 1             | 1               | 2              | 4         | At Risk |
| CUST005306  | 1             | 2               | 4              | 7         | At Risk |
| CUST005472  | 1             | 3               | 4              | 8         | Others  |
| CUST011758  | 1             | 1               | 1              | 3         | At Risk |
| CUST039281  | 1             | 2               | 5              | 8         | At Risk |
| CUST008089  | 1             | 2               | 4              | 7         | At Risk |

## Q8.Moving Average Revenue (7-day & 30-day)
```SQL
WITH daily_rev AS (
    SELECT 
        order_date::date AS sales_date,
        SUM(total_amount) AS daily_revenue
    FROM amazon
    GROUP BY 1
)
SELECT 
    sales_date,
    daily_revenue,
    ROUND(AVG(daily_revenue) OVER (ORDER BY sales_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)::numeric, 2) AS ma_7d,
    ROUND(AVG(daily_revenue) OVER (ORDER BY sales_date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW)::numeric, 2) AS ma_30d
FROM daily_rev
ORDER BY sales_date limit 5;
```
| sales_date | daily_revenue | ma_7d    | ma_30d   |
| ---------- | ------------- | -------- | -------- |
| 2020-01-01 | 99122.23      | 99122.23 | 99122.23 |
| 2020-01-02 | 82984.64      | 91053.44 | 91053.44 |
| 2020-01-03 | 76157.87      | 86088.25 | 86088.25 |
| 2020-01-04 | 114255.63     | 93130.09 | 93130.09 |
| 2020-01-05 | 83398.27      | 91183.73 | 91183.73 |

## Q9.Customer Lifetime Value (CLV) by Cohort
---
```SQL
WITH cohort AS (
    SELECT 
        customer_id,
DATE_TRUNC('month', MIN(order_date::timestamp)) AS cohort_month,
        SUM(total_amount) AS lifetime_value
    FROM amazon
    GROUP BY customer_id
)
SELECT 
    cohort_month,
    COUNT(DISTINCT customer_id) AS customers,
    ROUND(AVG(lifetime_value)::numeric, 2) AS avg_clv,
    ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY lifetime_value)::numeric, 2) AS p75_clv
FROM cohort
GROUP BY cohort_month
ORDER BY cohort_month;
```
| cohort_month        | customers | avg_clv | p75_clv |
| ------------------- | --------- | ------- | ------- |
| 2020-01-01 00:00:00 | 1697      | 5092.05 | 6931.94 |
| 2020-02-01 00:00:00 | 1415      | 4959.76 | 6800.44 |
| 2020-03-01 00:00:00 | 1560      | 5092.17 | 6906.46 |
| 2020-04-01 00:00:00 | 1531      | 5062.60 | 6966.26 |
| 2020-05-01 00:00:00 | 1459      | 4891.52 | 6765.01 |

## Q10.ABC Analysis (Product Classification by Revenue)
---
```SQL
WITH product_rev AS (
    SELECT 
        product_id,
        product_name,
        SUM(total_amount) AS revenue,
        SUM(total_amount) * 100.0 / SUM(SUM(total_amount)) OVER () AS cum_pct
    FROM amazon
    GROUP BY product_id, product_name
),
abc AS (
    SELECT 
        *,
        CASE 
            WHEN cum_pct <= 80 THEN 'A'
            WHEN cum_pct <= 95 THEN 'B'
            ELSE 'C'
        END AS abc_class
    FROM product_rev
)
SELECT abc_class, COUNT(*) AS products, SUM(revenue) AS total_revenue,
       100.0 * SUM(revenue) / SUM(SUM(revenue)) OVER () AS revenue_pct
FROM abc
GROUP BY abc_class;
```
| abc_class | products | total_revenue | revenue_pct |
| --------- | -------- | ------------- | ----------- |
| A         | 51       | 172559505     | 100         |

## Q11.Churn Analysis - Customers who didn't order in last 90 days
---
``` SQL
WITH last_order AS (
    SELECT 
        customer_id,
        MAX(order_date) AS last_order_dt,
        CURRENT_DATE - MAX(order_date)::date AS days_since_last_order
    FROM amazon
    GROUP BY customer_id
)
SELECT 
    COUNT(CASE WHEN days_since_last_order > 90 THEN 1 END) AS churned_customers,
    ROUND(100.0 * COUNT(CASE WHEN days_since_last_order > 90 THEN 1 END) / COUNT(*)::numeric, 2) AS churn_rate_pct
FROM last_order;
```
| churned_customers | churn_rate_pct |
| ----------------- | -------------- |
| 43230             | 100.00         |

## Q12.Window Function - Running Revenue by Category
---
```SQL
SELECT 
    category,
    order_date,
    SUM(total_amount) OVER (PARTITION BY category ORDER BY order_date) AS running_revenue,
    SUM(total_amount) OVER (PARTITION BY category ORDER BY order_date 
                            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_category_revenue
FROM amazon
ORDER BY category, order_date limit 5;
```
| category | order_date | running_revenue | total_category_revenue |
| -------- | ---------- | --------------- | ---------------------- |
| Books    | 2020-01-01 | 21789.97        | 28687489.6199999       |
| Books    | 2020-01-01 | 21789.97        | 28687489.6199999       |
| Books    | 2020-01-01 | 21789.97        | 28687489.6199999       |
| Books    | 2020-01-01 | 21789.97        | 28687489.6199999       |
| Books    | 2020-01-01 | 21789.97        | 28687489.6199999       |

## Q13.op 3 Brands by Revenue Share within Each Category
---
```SQL
WITH brand_rev AS (
    SELECT 
        category,
        brand,
        SUM(total_amount) AS revenue,
        RANK() OVER (PARTITION BY category ORDER BY SUM(total_amount) DESC) AS rank
    FROM amazon
    GROUP BY category, brand
)
SELECT * FROM brand_rev 
WHERE rank <= 3
ORDER BY category, revenue DESC limit 5;
```
| category | brand      | revenue    | rank |
| -------- | ---------- | ---------- | ---- |
| Books    | CoreTech   | 3067298.81 | 1    |
| Books    | FitLife    | 2983471.75 | 2    |
| Books    | UrbanStyle | 2930980.28 | 3    |
| Clothing | KiddoFun   | 2920106.01 | 1    |
| Clothing | UrbanStyle | 2916416.84 | 2    |

## Q14.Order-to-Ship Time Analysis (assuming shipping_cost > 0 implies shipped)
---
```SQL
SELECT 
    order_status,
    round(AVG(shipping_cost)::numeric,2) AS avg_shipping_cost,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_amount) AS median_order_value,
    COUNT(*) AS order_count
FROM amazon
WHERE shipping_cost > 0
GROUP BY order_status;
```
| order_status | avg_shipping_cost | median_order_value | order_count |
| ------------ | ----------------- | ------------------ | ----------- |
| Cancelled    | 7.39              | 725.56             | 5676        |
| Delivered    | 7.41              | 712.6              | 140249      |
| Pending      | 7.40              | 724.365            | 7718        |
| Returned     | 7.43              | 703.95             | 5698        |
| Shipped      | 7.41              | 722.93             | 28528       |

## Q15.Advanced: Pareto Principle Check (80/20 Rule)
---
``` SQL
WITH customer_rev AS (
    SELECT 
        customer_id,
        SUM(total_amount) AS revenue,
        SUM(total_amount) * 100.0 / SUM(SUM(total_amount)) OVER () AS running_pct
    FROM amazon
    GROUP BY customer_id
    ORDER BY revenue DESC
)
SELECT 
    COUNT(*) FILTER (WHERE running_pct <= 80) AS customers_contributing_80pct,
    ROUND(100.0 * COUNT(*) FILTER (WHERE running_pct <= 80) / COUNT(*), 2) AS pct_of_customers,
    '80/20 Rule Validation' AS analysis
FROM customer_rev;
```
| customers_contributing_80pct | pct_of_customers | analysis              |
| ---------------------------- | ---------------- | --------------------- |
| 43230                        | 100.00           | 80/20 Rule Validation |

## Q16.How many products were sold in each category?
---
``` SQL
SELECT 
    category,
    COUNT(*) AS total_orders,
    SUM(quantity) AS total_quantity_sold,
    SUM(total_amount) AS revenue
FROM amazon
GROUP BY category
ORDER BY revenue DESC;
```
| category          | total_orders | total_quantity_sold | revenue          |
| ----------------- | ------------ | ------------------- | ---------------- |
| Electronics       | 31683        | 95289               | 29260310.7800001 |
| Sports & Outdoors | 31564        | 94913               | 28807925.25      |
| Clothing          | 30930        | 93181               | 28714739.9699999 |
| Books             | 31490        | 93874               | 28687489.62      |
| Toys & Games      | 31029        | 93340               | 28562069.0300002 |
| Home & Kitchen    | 31210        | 93261               | 28526970.35      |

## Q17. Show top 5 selling products
```SQL
SELECT 
    product_name,
    SUM(quantity) AS total_sold,
    SUM(total_amount) AS total_revenue
FROM amazon
GROUP BY product_name
ORDER BY total_revenue DESC
LIMIT 5;
```
| product_name        | total_sold | total_revenue |
| ------------------- | ---------- | ------------- |
| Memory Card 128GB   | 11667      | 3617105.11    |
| LED Desk Lamp       | 11942      | 3613309.03    |
| Mechanical Keyboard | 11599      | 3589285.36    |
| Gaming Mouse        | 11679      | 3587031.21    |
| Electric Kettle     | 11551      | 3569803.01    |

## Q18. Which payment method is used the most?
```SQL
SELECT 
    payment_method,
    COUNT(*) AS usage_count
FROM amazon
GROUP BY payment_method
ORDER BY usage_count DESC;
```
| payment_method   | usage_count |
| ---------------- | ----------- |
| Credit Card      | 65917       |
| Debit Card       | 37547       |
| UPI              | 28319       |
| Amazon Pay       | 28207       |
| Net Banking      | 18654       |
| Cash on Delivery | 9262        |

## Q19. How many unique customers are there?
---
```SQL
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM amazon;
```
| total_customers |
| --------------- |
| 43230           |

## Q20. Which city did the order come from? (Unique city)
---
```SQL
SELECT DISTINCT city 
FROM amazon
ORDER BY city limit 5;
```
| city      |
| --------- |
| Austin    |
| Charlotte |
| Chicago   |
| Columbus  |
| Dallas    |
