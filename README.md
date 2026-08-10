# Data_Model_Project
Redesigned a messy 23-table Power BI import proper star schema — surrogate keys, shared dimensions, an accumulating snapshot fact for order tracking, and row-level security.

### The Problem
I imported 23-25 tables from source system directly into Power BI. The result was a flat, unstructured semantic model with 23 tables and 12 relationships — no clear separation between facts and dimensions, duplicate tables, and dead-end columns.
