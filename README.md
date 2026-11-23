# Zomato-Sales-Analysis
End-to-end Zomato sales analysis using MySQL and Python. Built a 4-table relational schema (60 restaurants, 500 customers, 6,000 orders), wrote 14 analytical SQL queries, and created pandas/seaborn visualizations. Found weekends drive 48% of orders, city rankings mostly reflect restaurant count, and discounts did not increase basket size.
# Zomato Sales Analysis (SQL + Python)

An end-to-end analysis of a Zomato-style food-delivery dataset: relational schema design, SQL analysis in MySQL, and exploratory analysis and visualization in Python.

> **Data note:** The dataset is **synthetic**, generated with `generate_data.py` (seeded, so it is reproducible). The pipeline and methods are real, but the specific findings below describe this generated data, not the real Zomato business. Where a pattern is an artifact of how the data was generated, it is flagged.

## Tech stack

- **MySQL** (Workbench): schema, constraints, analytical queries
- **Python**: pandas, matplotlib, seaborn, mysql-connector-python

## Project structure

```
zomato_project/
├── data/                        # restaurants, customers, orders, order_items (CSV)
├── generate_data.py             # synthetic data generator
├── zomato_sales_mysql.sql       # MySQL schema + data (import this first)
├── analysis_mysql.sql           # 14 business-analysis queries
├── python_analysis_mysql.py     # pandas analysis + 10 charts (reads from MySQL)
├── charts/                      # generated visualizations
└── README.md
```

## Dataset

| Table | Rows | Description |
|---|---|---|
| `restaurants` | 60 | City, cuisine, cost for two, rating, votes, online-order flag |
| `customers` | 500 | City, signup date |
| `orders` | 6,000 | Date, payment method, discount, amount, delivery time, rating (Jan 2024 – Dec 2025) |
| `order_items` | 18,173 | Item, category, price, quantity per order |

## How to run

1. Import `zomato_sales_mysql.sql` in MySQL Workbench (File → Open SQL Script → run).
2. Run the queries in `analysis_mysql.sql`.
3. `pip install mysql-connector-python pandas matplotlib seaborn`
4. Edit `DB_CONFIG` in `python_analysis_mysql.py`, then `python3 python_analysis_mysql.py`.

## Key findings

**Overall:** 6,000 orders generated ₹44.16 lakh in revenue, an average order value (AOV) of ₹736. Revenue was ₹21.7L in 2024 and ₹22.5L in 2025. Total discounts given: ₹3.48L (about 7.3% of gross sales). Average delivery time was 40 minutes (range 15–65).

**1. Weekends drive demand, and Saturday dominates.** Saturday and Sunday account for 48.3% of orders despite being 2 of 7 days. Saturday alone has 2,042 orders, about 3.3x the average weekday (~620). Sunday (857) is much weaker than Saturday. *Recommendation:* concentrate delivery capacity and promotions on Saturdays; test Sunday-specific offers.

**2. Cuisine and city leaders are driven by supply, not demand per outlet.**
- Italian leads cuisine revenue (₹5.67L) and Mumbai leads city revenue (₹7.47L).
- But Italian has the most restaurants (8) and Mumbai has the most (10). Normalized per restaurant, Mumbai drops out of the top 4 (Kolkata, Delhi and Jaipur lead at roughly ₹77–79K per restaurant).
- *Takeaway:* total revenue rankings mostly reflect how many outlets a city or cuisine has. Revenue per restaurant is the fairer performance metric.

**3. Best sellers.** Chicken 65 is the top item by revenue (₹3.11L, 1,296 units), followed by Butter Chicken and Chocolate Brownie. By volume, breads lead (Garlic Naan 2,521 units, Butter Naan 2,478) because they are cheap add-ons; the Chocolate Brownie (1,851 units) stands out as a dessert that sells in high volume. By category, Main Course (₹17.7L) and Starters (₹13.1L) lead; Desserts contribute ₹8.2L.

**4. Discounts reduce revenue per order mechanically, not behaviorally.**
Average order value falls from ₹791 (no discount) to ₹637 (20% discount). It is tempting to read this as "discounted customers order less," but the *pre-discount* basket size is flat across discount levels (about ₹786–806, correlation 0.005). The drop in AOV is simply the discount being subtracted. In this data, discounts did **not** increase basket size, and about 57% of orders carried some discount. *Recommendation:* a real discount-effectiveness test needs a control group and order-frequency data, which this dataset can't provide.

**5. Restaurant rating does not predict revenue.** The correlation between a restaurant's rating and its revenue is 0.02. This is expected here because ratings were assigned independently of order volume; in real data this relationship would be worth investigating.

**6. Payment mix is balanced.** All five methods sit between 19.4% and 20.5% of orders (Wallet and Cash on Delivery slightly ahead), so there is no dominant payment channel.

**7. Customers are broadly distributed.** All 500 customers placed between 3 and 23 orders. The top 10% of customers (50 people) contribute only 16% of revenue, so there is no heavy dependence on a few big spenders, and retention work should target the whole base rather than a VIP segment.

**8. Customer satisfaction.** Of orders with a rating (79.9%), 2,962 of 4,795 (61.8%) were rated 4 or 5, and the average rating is 3.59. About 26% of rated orders were 1–2 stars, which is a meaningful complaint share worth analyzing against delivery time in a real dataset.

## Limitations

- **Synthetic data.** Order dates were sampled from a bell-shaped distribution, so the revenue curve (rising through 2024, peaking Nov 2024, then falling sharply through late 2025) is an artifact of the generator, not a real seasonal trend. Do not read the late-2025 decline as genuine churn.
- Ratings, discounts and payment methods were assigned randomly and independently, which is why most correlations are near zero.
- No repeat-order time gaps, cohort retention, or delivery-city vs restaurant-city logic, which a real dataset would support.

## Possible extensions

- Customer cohort and retention analysis (needs realistic ordering behavior)
- RFM segmentation
- Delivery time vs rating analysis
- A Power BI or Tableau dashboard on top of the MySQL database
- Swap in a real Zomato dataset (e.g., from Kaggle); the schema and queries transfer with small column mappings
