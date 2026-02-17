# Brazilian Ecommerce Project
A full data pipeline built on the Olist Brazilian E-Commerce dataset from Kaggle. Raw CSV data across 9 files was imported into PostgreSQL, cleaned and transformed using advanced SQL, then modelled and visualised in Power BI.

## SQL Transformation Highlights

The transformation script `CleanEcommerceData.sql` demonstrates:

- **CTEs** to break complex multi-source logic into readable, auditable steps
- **`NTILE(4)` window function** for seller revenue quartile banding — replaces the correlated subquery workaround required in SQLite
- **`ROW_NUMBER()` window function** to identify the primary payment method per order in a single table scan, without correlated subqueries
- **`PERCENTILE_CONT(0.5)`** for true median imputation of NULL product dimensions — more statistically robust than using averages on skewed data
- **`EXTRACT()` and `TO_CHAR()`** for clean timestamp decomposition into year, month, quarter and day name
- **EPOCH-based delivery durations** — `EXTRACT(EPOCH FROM (ts2 - ts1)) / 86400` for precise day-level delivery calculations
- **`SUM() FILTER (WHERE ...)`** to pivot payment types into columns — cleaner and faster than `CASE WHEN` aggregations
- **Deduplication** of ~1M geolocation rows down to ~19k representative zip-level coordinates via `AVG(lat/lng) GROUP BY zip`
- **Data quality flags** — `date_logic_error_flag` surfaces orders where delivery timestamp precedes purchase timestamp

---

## Report Pages & Key Insights

### Page 1 — Executive Overview

> High-level business health across the full dataset

| KPI | Value |
|-----|-------|
| Total Revenue | R$ 15.84M |
| Total Orders | 99K |
| Avg Order Value | R$ 160.58 |
| Avg Review Score | 4.03 / 5 |
| Late Delivery Rate | 7.85% |

**Key Insights:**

- **Credit card dominates** at 78.44% of total revenue (R$ 12.43M), with boleto — a Brazilian bank slip payment method used by consumers without traditional card access — accounting for a further 17.94% (R$ 2.84M). This reflects Brazil's unique payments landscape and is a meaningful data point for any market entry analysis
- **96K out of 99K orders reached Completed status**, representing a ~97% fulfilment rate — a strong operational baseline
- The **7.85% late delivery rate represents approximately 7,800 orders** that missed their estimated delivery window. As Page 4 shows, late deliveries are the single biggest driver of negative reviews — making this the highest-priority operational metric in the dataset

---

### Page 2 — Customer & Geography

> Revenue and order concentration across Brazil's 26 states

**Key Insights:**

- **São Paulo state dominates with R$ 5.92M** in revenue — nearly 3× more than Rio de Janeiro (R$ 2.13M) in second place, reflecting Brazil's extreme economic concentration in the south-east
- **São Paulo city alone generated R$ 2.17M**, more than the entire states of Bahia, Santa Catarina, and Distrito Federal combined
- The filled map reveals a striking **north-south revenue divide**: south-eastern states (SP, RJ, MG) drive the overwhelming majority of revenue, while the vast northern states (Amazonas, Pará, Acre) show minimal activity — pointing to either significant untapped market potential or last-mile logistics barriers in the north
- **Minas Gerais (R$ 1.86M)** and **Rio Grande do Sul (R$ 0.89M)** are the standout performers outside the São Paulo / Rio axis and represent the most viable expansion targets

---

### Page 3 — Product Performance

> Category-level revenue and product data quality

**Key Insights:**

- **Health & Beauty** is the top revenue category at R$ 1.44M, followed by **Watches & Gifts** (R$ 1.31M) and **Bed, Bath & Table** (R$ 1.24M) — the top 3 are all lifestyle and discretionary purchases, not electronics or essentials as might be expected for an emerging e-commerce market
- **Small and Large size tier products** generate the most revenue (R$ 4.5M and R$ 4.3M respectively), while Extra Small products (R$ 0.5M) significantly underperform — suggesting the platform is optimised for mid-to-large physical goods
- The **Price vs Review Score scatter plot shows no meaningful positive correlation** — higher priced items do not reliably generate better reviews, meaning product quality and delivery experience matter more to customers than price point alone

---

### Page 4 — Delivery & Seller Performance

> Operational efficiency and the link between logistics and customer satisfaction

**Key Insights:**

- **The standout finding of the entire dataset:** review scores show a clear, consistent decline as delivery speed worsens. Express deliveries (1–3 days) achieve the highest review scores, while Very Slow deliveries (30+ days) score lowest — confirming that delivery speed is the primary driver of customer satisfaction on the platform
- **89K orders arrived early vs only 8K late** — yet that 8K generates a disproportionate share of negative reviews given the score drop associated with late delivery. Reducing late deliveries is the single highest-leverage improvement available to Olist
- **Freight cost as a % of item price shows no strong correlation with review score** — customers are significantly less sensitive to what they pay for shipping than they are to how fast it arrives. Speed matters more than price
- The **seller revenue quartile analysis** confirms a heavy concentration of revenue in the top 25% of sellers, which is typical of marketplace dynamics but highlights platform dependency risk

---

## Data Model

```
dim_customers ──────────────────────────── dim_products
(customer_id PK)                           (product_id PK)
customer_city                              product_category_english
customer_state_full                        product_size_tier
latitude / longitude                       product_weight_tier
has_geolocation                            product_volume_cm3
       │ Many-to-One                       Many-to-One │
       │                                               │
       └──────────────── fact_orders ─────────────────┘
                         (order_id, order_item_id)
                         customer_id FK
                         product_id FK
                         seller_id
                         purchase_date
                         order_status_group
                         actual_delivery_days
                         delivery_timeliness
                         delivery_speed_tier
                         item_price / freight_value / item_total
                         primary_payment_method
                         avg_review_score
                         review_sentiment
                         seller_revenue_quartile
```

---

## Dataset

**Source:** [Olist Brazilian E-Commerce Public Dataset — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

**Coverage:** ~100,000 orders placed between 2016 and 2018 across Brazil

| Table | Rows |
|-------|------|
| olist_orders_dataset | ~99,441 |
| olist_order_items_dataset | ~112,650 |
| olist_order_payments_dataset | ~103,886 |
| olist_order_reviews_dataset | ~99,224 |
| olist_customers_dataset | ~99,441 |
| olist_products_dataset | ~32,951 |
| olist_sellers_dataset | ~3,095 |
| olist_geolocation_dataset | ~1,000,163 |
| product_category_name_translation | 71 |

---

*Dataset provided by Olist, the largest department store in Brazilian marketplaces.*
