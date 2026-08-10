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
