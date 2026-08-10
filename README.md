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

Before touching a single table, I worked through three questions using the raw import itself.

**1. Analyze the business**

This is retail/e-commerce order data — customers placing orders, orders getting shipped and invoiced, campaigns driving sales, and payments coming in. The reporting needs to answer questions like: what are we selling, to whom, through which channel, and is a campaign actually driving revenue. That's the lens I used to decide what mattered and what didn't.

**2. Explore the process**

Traced the real sequence behind the tables instead of trusting how they were named:
- **Order lifecycle:** order placed → shipped (`shipments`) → invoiced (`INVOICES`) → paid (`payments`)
- **Sales flow:** customer (`CUST_MASTER`, `customer_contacts`) → order (`ORDERS_2025`/`ORDERS_2026`) → order lines (`order_line_items`, `invoice_lines`) → product (`products`, `subcategories`)
- **Marketing flow:** campaign (`CAMPAIGN_LOG`, `campaign_skus`) → promotion applied to a product → sale

This is what told me `fact_order_process` needed to be an accumulating snapshot (one row per order, a column per milestone date) instead of five separate fact tables joined on order ID.

**3. Understand the data**

Opened the raw tables row by row before deciding anything:
- `ORDERS_2025` and `ORDERS_2026` had identical structure — same columns, just split by year for no reason tied to the business
- `security`, `regions`, `cities`, `subcategories`, and `campaign_skus` all had generic `Column1`/`Column2` headers — had to trace each back to source to find out `security.Column1` was actually `region` and `Column2` was `user_email`
- `Address`, `cities`, and `regions` were three separate tables all describing the same thing: geography
- `Sheet1` had no relationships to anything else — a leftover from the original import, not real data

That inventory is what made Phase 2 (building the dimensions) possible — I already knew which tables belonged together before I started merging anything.
