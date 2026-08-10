# Data_Model_Project
Redesigned a messy 23-table Power BI import proper star schema — surrogate keys, shared dimensions, an accumulating snapshot fact for order tracking, and row-level security.

### 1) The Problem
I imported 23-25 tables from source system directly into Power BI. The result was a flat, unstructured semantic model with 23 tables and 12 relationships — no clear separation between facts and dimensions, duplicate tables, and dead-end columns.
![Data Model Before](https://raw.githubusercontent.com/ompatil05/Data_Model_Project/main/images/01_model_before.png)

### 2) Issues in the Model
i) Duplicate tables split by year, instead of one table with a date column. 
  example: ORDERS_2025, ORDERS_2026

ii) Inconsistent naming conventions 

iii) Redundant tables covering the same entity , Address, cities, regions all describing geography separately

iv) No surrogate keys: Joins relied on inconsistent natural keys across tables

v) No fact/dimension distinction :Transactional data (orders, payments, campaigns) and descriptive data (customers, products, geography) lived at the same flat level
