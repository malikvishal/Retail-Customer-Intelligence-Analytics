# Retail-Customer-Intelligence-Analytics
A mid-sized Indian retail chain operating across 10 stores (metro + tier-2 cities) and two online channels approached me to help them understand why customer retention was inconsistent across locations and why certain marketing campaigns were not translating into repeat purchases.

## 📌 Project Background

A mid-sized Indian retail chain operating across 10 stores (metro + tier-2 cities) and two online channels approached me to help them understand why customer retention was inconsistent across locations and why certain marketing campaigns were not translating into repeat purchases.

The client had raw transactional data sitting in spreadsheets with no structured analysis in place. My role was to build a complete analytics workflow from scratch — from data cleaning and modelling to SQL-based business analysis, an interactive Power BI dashboard, and a final set of strategic recommendations presented to the leadership team.

**Core business question the client wanted answered:**
> *"Which customers are most valuable, which stores are underperforming, and are our marketing campaigns actually driving loyalty — or just one-time purchases?"*

---

## 🗂️ Dataset Overview

| Table | Rows | Description |
|---|---|---|
| `customer` | 3,900 | Purchase transactions with demographics, spend, shipping, discount, subscription |
| `products` | 25 | Product catalogue with cost price, selling price, profit margin, stock levels |
| `stores` | 10 | Store details — city, tier, region, store type, size in sqft |
| `marketing_campaigns` | 8 | Seasonal campaigns with budget, channel, target category, discount offered |

**Key columns across tables:**
- Customer demographics: Age, Gender, Location, Subscription Status
- Purchase details: Item, Category, Amount (USD), Season, Size, Color
- Behaviour: Discount Applied, Previous Purchases, Frequency, Review Rating, Shipping Type
- Store attributes: City Tier (Metro/Tier-2/Online), Region, Store Size sqft
- Campaign attributes: Season, Channel, Budget (INR Lakhs), Discount %, Target Category

---

## 🔧 Project Workflow

```
Raw Data (CSV)
     │
     ▼
Python — EDA & Data Preparation
     │
     ▼
SQL Server — Business Analysis (25 queries across 4 tables)
     │
     ▼
Power BI — 3-Tab Interactive Dashboard + DAX Measures
     │
     ▼
Report & Presentation — Delivered to Client
```

---

## 🐍 Phase 1 — Python: Data Preparation & EDA

**Notebook:** `Customer_Shopping_Behavior_Analysis.ipynb`

Key steps performed:

**Data Cleaning:**
- Imported dataset using `pandas` and inspected structure with `df.info()` and `df.describe()`
- Identified and imputed 37 missing values in `review_rating` using **category-level median** (more accurate than global median)
- Renamed all columns to snake_case for consistency and SQL compatibility
- Dropped `promo_code_used` after confirming 100% overlap with `discount_applied` (redundant column)

**Feature Engineering:**
- Created `age_group` by binning customer ages into: Young Adult / Adult / Middle-aged / Senior
- Created `purchase_frequency_days` by mapping frequency labels to numeric intervals (Weekly=7, Monthly=30, etc.)
- Added `store_id` (FK → stores) with weighted random assignment reflecting real metro vs tier-2 traffic distribution
- Added `campaign_id` (FK → marketing_campaigns) based on season and discount logic — 37% purchases marked as organic (NULL)

**New Tables Generated:**
- `products` — 25 products with cost price, selling price, profit margin %, stock quantity, reorder level, supplier
- `stores` — 10 stores across Mumbai, Delhi, Bengaluru, Hyderabad, Chennai, Pune, Jaipur, Ahmedabad + 2 online
- `marketing_campaigns` — 8 seasonal campaigns (Diwali, Summer, Monsoon, Winter etc.) with budgets in INR Lakhs

**Data Quality Checks before DB load:**
- Referential integrity: all `store_id` and `campaign_id` values validated against master tables
- No duplicate `customer_id` values
- No negative profit margins
- Online stores correctly have NULL `store_size_sqft`

**Database Load:**
- Connected to SQL Server using `SQLAlchemy` + `pyodbc` with `fast_executemany=True`
- Loaded all 4 tables into SQL Server for analysis

---

## 🗃️ Phase 2 — SQL Server: Business Analysis

**File:** `customer_behavior_sql_queries_sqlserver.sql`

25 queries across 5 sections answering key business questions:

### Section A — Core Customer Analysis (Q1–Q10)
| # | Question | Key Finding |
|---|---|---|
| Q1 | Revenue by gender | Male customers generated 2x revenue ($157,890 vs $75,191) |
| Q2 | High-spend discount users | 839 customers used discounts but spent above average |
| Q3 | Top 5 products by rating | Gloves (3.86), Sandals (3.84), Boots (3.82) |
| Q4 | Shipping type vs spend | Express ($60.48) vs Standard ($58.46) avg order value |
| Q5 | Subscriber vs non-subscriber spend | Similar AOV ($59.49 vs $59.87) — subscription drives volume not spend |
| Q6 | Discount-dependent products | Hat (50%), Sneakers (49.66%), Coat (49.07%) |
| Q7 | Customer segmentation | Loyal: 3,116 (79.9%), Returning: 701, New: 83 |
| Q8 | Top 3 products per category | Clothing: Blouse/Pants/Shirt; Footwear: Sandals/Shoes/Sneakers |
| Q9 | Repeat buyers + subscriptions | 2,518 repeat buyers not subscribed — largest retention opportunity |
| Q10 | Revenue by age group | Young Adults lead ($62,143), Seniors lowest ($55,763) |

### Section B — Products Analysis (Q11–Q14)
- Margin-squeeze products: high volume items with <40% margin
- Inventory risk: items below reorder level
- Revenue gap: estimated full-price revenue vs actual (discount impact)
- Supplier quality: avg rating and revenue by supplier

### Section C — Store Performance (Q15–Q18)
- Revenue and avg order value by city tier
- Regional revenue + subscription rate comparison
- **Revenue per sqft** by physical store (key retail KPI)
- Online vs offline: avg rating and discount usage

### Section D — Campaign Analysis (Q19–Q22)
- Campaign ROI: revenue generated vs budget spent
- Channel performance: Email vs SMS vs Social Media
- Festive vs organic customer loyalty (repeat purchase rate)
- Discount uptake rate per campaign

### Section E — Advanced Multi-table (Q23–Q25)
- Full funnel: campaign performance broken down by store
- High-value customer identification (subscribed + loyal + above-avg spend)
- Running revenue total by store using `SUM() OVER()` window function

---

## 📊 Phase 3 — Power BI Dashboard (3 Tabs)

**File:** `customer_behavior_dashboard.pbix`

**Relationships configured in Model View:**
- `customer[store_id]` → `stores[store_id]` (Many-to-One)
- `customer[campaign_id]` → `marketing_campaigns[campaign_id]` (Many-to-One)
- `customer[item_purchased]` → `products[product_name]` (Many-to-One)

### Tab 1 — Customer Overview
KPIs: Total Customers, Avg Purchase Amount, Avg Review Rating, Loyal Customer %
Visuals: Subscription donut, Revenue & Sales by category, Customer segment by age group (stacked bar), Purchase frequency distribution

### Tab 2 — Store & Product Performance
KPIs: Total Stores, Avg Profit Margin %, Online Revenue Share %, Items Below Reorder Level
Visuals: Revenue by store (ranked bar), Revenue per sqft (physical stores), Online vs Offline comparison, Metro vs Tier-2 subscription rate, **Profit margin vs volume scatter plot**, Inventory risk table

### Tab 3 — Campaign & Revenue Intelligence
KPIs: Total Budget, Campaign Revenue, Organic Revenue %, Best ROI Campaign
Visuals: Campaign ROI column chart (with break-even line), Revenue by channel, Campaign vs organic loyalty, Discount uptake by campaign, Campaign-season revenue overlay (attribution visual)

**DAX highlights:**
- `Revenue Per Sqft` — uses `MAX()` on store sqft to avoid row repetition after join
- `Revenue Gap %` — uses `SUMX()` row iterator for cross-table margin calculation
- `Revenue Lift % vs Organic` — attribution metric comparing campaign vs organic AOV

---

## 💡 Key Business Findings & Recommendations

### Finding 1 — Subscription programme is underutilised
Only 27% of customers are subscribed despite subscribers having similar AOV. More critically, **2,518 repeat buyers (>5 purchases) are not subscribed** — the single largest retention opportunity.

**Recommendation:** Launch a targeted re-engagement campaign for repeat non-subscribers offering exclusive benefits (free shipping, early access). Priority: Young Adult and Middle-aged segments who show the highest purchase frequency.

### Finding 2 — Online channels dominate but physical stores are more efficient
Online stores (S09, S10) account for ~41% of revenue but physical stores show higher **revenue per sqft** efficiency, particularly Bengaluru Koramangala and Mumbai Linking Road.

**Recommendation:** Invest in expanding high-performing physical locations rather than acquiring new ones. Tier-2 stores (Pune, Jaipur) show strong subscription rates relative to their size — worth targeted local campaigns.

### Finding 3 — Festive campaigns drive genuine loyalty; Summer campaigns do not
Customers acquired via Festive campaigns (Diwali, Navratri) show significantly higher average previous purchases than those acquired via Summer or generic campaigns — indicating genuine loyalty lift. The Summer Social Media campaign showed near-zero revenue lift over organic baseline, suggesting budget was not adding value.

**Recommendation:** Reallocate Summer Social Media budget toward Festive Email campaigns which show the highest ROI and long-term loyalty metrics.

### Finding 4 — Several high-volume products are margin-squeezed
Products like Jeans (36.6%), Shorts (36.6%), and Sandals (34.9%) have the lowest profit margins but are among the top sellers by volume. Discounts are being applied to these products at rates above 45%.

**Recommendation:** Reduce discount frequency on low-margin products. Use dynamic pricing or bundle offers instead to protect margins without losing volume.

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Excel | Data Profiling, Business Analysis |
| Python (pandas, numpy, sqlalchemy) | Data cleaning, feature engineering, DB load |
| SQL Server (T-SQL) | Business analysis — 25 queries, CTEs, window functions |
| Power BI + DAX | 3-tab interactive dashboard, 30+ measures |
| pyodbc | SQL Server connection from Python |
| GitHub | Version control and project documentation |


---

## 👤 About

Built as part of a freelance Business data Intelligence engagement for an Indian retail client.


