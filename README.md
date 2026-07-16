# OList Data Pipeline

This repository implements a medallion-style data pipeline for the OList Brazilian e-commerce dataset using Databricks notebooks, Spark, SQL, and Delta tables.

## Dataset

- Source: Kaggle — [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Raw files expected at: `/Volumes/workspace/default/olist/`

## Data Model

### Star schema (Gold layer target)

<img src="https://github.com/user-attachments/assets/dbff6681-65c2-4c74-9626-5788ecc0f1a0" alt="Star schema" style="padding: 12px; border-radius: 8px;" />

### Raw dataset relations

<img src="https://github.com/user-attachments/assets/84b58ac5-f619-49d2-a345-9d15db729036" alt="Raw dataset relations" style="padding: 12px; border-radius: 8px;" />

## Repository Notebooks

- `Data Exploration.ipynb`  
  Explores source tables, key candidates, duplicates, and relationship behavior.

- `Data Quality Check.ipynb`  
  Validates referential integrity across layers and documents quality actions.

- `Bronze Layer.ipynb`  
  Ingests raw CSV files into Delta tables under the `bronze` schema.

- `Silver Layer.ipynb`  
  Builds conformed dimensions and fact table under the `silver` schema.

- `Gold Layer.ipynb`  
  Publishes analytics-friendly views under the `gold` schema.

## Pipeline Summary

### Bronze layer

Creates and loads these raw Delta tables:

- `bronze.customers`
- `bronze.geolocation`
- `bronze.order_items`
- `bronze.order_payments`
- `bronze.order_reviews`
- `bronze.orders`
- `bronze.products`
- `bronze.sellers`
- `bronze.product_category_translation`

### Silver layer

Creates dimensional model tables with surrogate keys and `dwh_date` defaults:

- Dimensions: `silver.dim_customers`, `silver.dim_sellers`, `silver.dim_products`, `silver.dim_reviews`, `silver.dim_payments`
- Fact: `silver.fact_orders`

Key transformation patterns:

- Null handling with `coalesce` for payment and product attributes
- Type normalization with `try_cast` for review scores and date/timestamp columns
- Review filtering to keep only reviews linked to existing orders and valid timestamps
- Fact table assembly through joins between bronze transaction data and silver dimensions

### Gold layer

Creates curated views with business-friendly naming:

- `gold.dim_customers`
- `gold.dim_sellers`
- `gold.dim_products`
- `gold.dim_reviews`
- `gold.dim_payments`
- `gold.fact_orders`

## Data Quality Findings Captured in Notebooks

- Some customer ZIP prefixes do not exist in geolocation data
- Geolocation contains repeated ZIP rows
- Orders, payments, items, sellers, and products are largely well-mapped
- Reviews include orphaned records; invalid/unmapped reviews are excluded in silver transformations

## Execution Order

Run notebooks in this order:

1. `Bronze Layer.ipynb`
2. `Silver Layer.ipynb`
3. `Gold Layer.ipynb`

## Notes

- Notebooks are designed for Databricks `%sql` + PySpark execution.
- Schema and object names follow medallion conventions: `bronze` → `silver` → `gold`.
