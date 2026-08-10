# Data_Model_Project
Redesigned a messy 23-table Power BI import proper star schema — surrogate keys, shared dimensions, an accumulating snapshot fact for order tracking, and row-level security.

### 1) The Problem
I imported 23-25 tables from source system directly into Power BI. The result was a flat, unstructured semantic model with 23 tables and 12 relationships — no clear separation between facts and dimensions, duplicate tables, and dead-end columns.
![Data Model Before](https://raw.githubusercontent.com/ompatil05/Data_Model_Project/main/images/01_model_before.png)

### 2) Issues in the Model
i) **<ins>Duplicate tables split by year</ins>**, instead of one table with a date column. 
  example: ORDERS_2025, ORDERS_2026

ii) **<ins>Inconsistent naming conventions</ins>**:Mix of Title Case (INVOICES), ALL_CAPS (CUST_MASTER), and snake_case (order_line_items)

iii) **<ins>Redundant tables covering the same entity</ins>**: Address, cities, regions all describing geography separately

iv) **<ins>No surrogate keys</ins>**: Joins relied on inconsistent natural keys across tables

v) **<ins>No fact/dimension distinction</ins>** :Transactional data (orders, payments, campaigns) and descriptive data     (customers, products, geography) lived at the same flat level


## Phase 1 — Explore Before You Build

Before touching any table, I worked through three questions against the raw import.

**1. Analyze the business**
This is retail/e-commerce order data — customers, orders, shipments, invoices, payments, and campaigns. Reporting needs to answer: what's selling, to whom, through which channel, and which campaigns actually drive revenue.

**2. Explore the process**
Traced the real business flow instead of trusting table names:
- Order lifecycle: order → shipped → invoiced → paid
- Sales flow: customer → order → order lines → product
- Marketing flow: campaign → promotion → sale

This is what led to designing `fact_order_process` as an accumulating snapshot (one row per order, one column per milestone) instead of five separate fact tables.

**3. Understand the data**
Reviewed each raw table before deciding anything:
- `ORDERS_2025` / `ORDERS_2026` — identical structure, split by year for no real reason
- `security`, `regions`, `cities`, `subcategories`, `campaign_skus` — generic `Column1`/`Column2` headers requiring tracing back to source
- `Address`, `cities`, `regions` — three tables describing one entity: geography
- `Sheet1` — unconnected leftover from the original import

This inventory is what made the dimension design in Phase 2 possible.

---

## Phase 2 — Dimensions: The Dimension Loop

Same three moves, repeated for every dimension.

**1. Collect**
Gathered every raw table describing one entity. Example: `Address`, `cities`, and `regions` all describe geography — one entity split across three tables.

**2. Combine**
Merged them into a single dimension. `Address` + `cities` + `regions` → `dim_geo`. Same pattern applied to customers (`CUST_MASTER`, `customer_contacts`, `Address` → `dim_customers`), products (`products`, `subcategories` → `dim_products`), and campaigns (`CAMPAIGN_LOG`, `campaign_skus` → `dim_campaign`).

**3. Clean**
Applied the standard to each merged table:
- `snake_case` naming
- `dim_` prefix
- `_key` surrogate key (e.g. `geo_key`, `product_key`)
- Dropped columns that didn't earn their place (leftover source artifacts, unused fields)

Repeating this loop one entity at a time — instead of merging everything at once — meant each dimension could be validated on its own before it was wired into a fact table.

---

## Phase 3 — Facts: Build, Connect, Test

A fact table always sits at the level of what you measure — so the first question for every fact was: where do the numbers come from?

**1. Pick an event, read its grain**
Before building, I stated out loud what one row represents — e.g. "one row in `fact_sales` = one order line." Skipping this is how double-counted totals happen.

**2. Build the fact from the details**
Merged `ORDERS_2025` + `ORDERS_2026` + `INVOICES` + `order_line_items` into grain-consistent fact tables: `fact_sales`, `fact_order_process`.

**3. Connect every dimension**
Wired each fact to `dim_customers`, `dim_products`, `dim_geo`, `dim_date`, `dim_campaign` — single-direction, 1-to-many, no bidirectional or many-to-many joins.

**4. Test so the numbers never break**
Recalculated Total Sales, Total Orders, and Active Customers against the raw source after every merge.

Two fact patterns worth calling out:
- **Factless fact** (`fact_promotion_coverage`) — no measures, just keys and a date range. Answers "was this product being promoted on this date," not a numeric question.
- **Accumulating snapshot fact** (`fact_order_process`) — one row per order, a column for every milestone (ordered → shipped → delivered → invoiced → paid), instead of five separate fact tables joined by order ID.

---

## Phase 4 — Polish: Secure and Validate

1. **Reconciled the standards** across every table
2. **Added `dim_date`** so every fact supports time intelligence
3. **Centralized measures** into a dedicated `_measure` table, kept separate from any physical table
4. **Added row-level security** — connected the `security` table (`region`, `user_email`) into the model. Not forced onto every fact: over-securing breaks the grain of facts that don't carry a region themselves.
5. **Final validation** — tested RLS as an actual user would see it (a "region = North America" user gets a different Total Sales than a "region = Europe" user), rather than trusting the relationship by design.


