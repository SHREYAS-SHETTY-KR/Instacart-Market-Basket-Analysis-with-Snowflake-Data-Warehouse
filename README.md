# Instacart Market Basket Analysis with Snowflake Data Warehouse

## Overview

This project demonstrates how to build a data warehouse using Snowflake for the Instacart Market Basket Analysis dataset. The dataset is sourced from Kaggle and stored in an S3 bucket. The data is then loaded into Snowflake tables, which are used to create dimensional and fact tables for analytical queries.

## Project Structure

1. **Data Storage**: Store the Kaggle dataset in an S3 bucket.
2. **Snowflake Stage and File Format**: Create a stage and define the file format in Snowflake.
3. **Data Loading**: Load data from the S3 bucket into Snowflake tables.
4. **Dimensional Modeling**: Build dimension and fact tables.
5. **Analytics**: Perform SQL queries to extract meaningful insights.

## Prerequisites

- Snowflake account
- AWS account with S3 bucket
- Kaggle account to download the dataset

## Dataset

The dataset can be found on Kaggle: [Instacart Market Basket Analysis](https://www.kaggle.com/competitions/instacart-market-basket-analysis/data)

## Steps to Reproduce

### 1. Data Storage in S3

First, download the dataset from Kaggle and upload the CSV files to an S3 bucket.

### 2. Snowflake Stage and File Format

Create a stage and define a file format in Snowflake to load data from the S3 bucket.

```sql
CREATE STAGE my_stage
URL = 's3://snowflake-project-data-shrey/Instacart/'
CREDENTIALS = (AWS_KEY_ID = 'YOUR_AWS_KEY_ID', AWS_SECRET_KEY = 'YOUR_AWS_SECRET_KEY');

CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"';
```

### 3. Data Loading into Snowflake Tables

Create tables in Snowflake and load data from the S3 bucket.

Aisles Table
```
CREATE TABLE aisles (
    aisle_id INTEGER PRIMARY KEY,
    aisle VARCHAR
);

COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

Departments Table
```
CREATE TABLE departments (
    department_id INTEGER PRIMARY KEY,
    department VARCHAR
);

COPY INTO departments (department_id, department)
FROM @my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

Products Table
```
CREATE OR REPLACE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR,
    aisle_id INTEGER,
    department_id INTEGER
);

COPY INTO products (product_id, product_name, aisle_id, department_id)
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

Orders Table
```
CREATE OR REPLACE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    eval_set STRING,
    order_number INTEGER,
    order_dow INTEGER,
    order_hour_of_day INTEGER,
    days_since_prior_order INTEGER
);

COPY INTO orders (order_id, user_id, eval_set, order_number, order_dow, order_hour_of_day, days_since_prior_order)
FROM @my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```
Order Products Table

```CREATE OR REPLACE TABLE order_products (
    order_id INTEGER,
    product_id INTEGER,
    add_to_cart_order INTEGER,
    reordered INTEGER,
    PRIMARY KEY (order_id, product_id)
);

COPY INTO order_products (order_id, product_id, add_to_cart_order, reordered)
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

### 4. Building Dimensional and Fact Tables

![Screenshot (39)](https://github.com/SHREYAS-SHETTY-KR/Instacart-Market-Basket-Analysis-with-Snowflake-Data-Warehouse/assets/79562771/02aa4696-64e8-44e4-a366-69eab1cb9190)


#### Create dimension and fact tables for analytical queries.
```
CREATE OR REPLACE TABLE dim_users AS (
  SELECT user_id FROM orders
);

CREATE OR REPLACE TABLE dim_products AS (
  SELECT product_id, product_name FROM products
);

CREATE OR REPLACE TABLE dim_aisles AS (
  SELECT aisle_id, aisle FROM aisles
);

CREATE OR REPLACE TABLE dim_departments AS (
  SELECT department_id, department FROM departments
);

CREATE OR REPLACE TABLE dim_orders AS (
  SELECT order_id, order_number, order_dow, order_hour_of_day, days_since_prior_order FROM orders
);

CREATE TABLE fact_order_products AS (
  SELECT
    op.order_id,
    op.product_id,
    o.user_id,
    p.department_id,
    p.aisle_id,
    op.add_to_cart_order,
    op.reordered
  FROM
    order_products op
  JOIN
    orders o ON op.order_id = o.order_id
  JOIN
    products p ON op.product_id = p.product_id
);
```

### 5. Analytics
#### Run the following SQL queries to extract insights from the data:

#### Total Number of Products Ordered per Department
```
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_departments d ON fop.department_id = d.department_id
GROUP BY
  d.department;
```

#### Top 5 Aisles with Highest Number of Reordered Products
```
SELECT
  a.aisle,
  COUNT(*) AS total_reordered
FROM
  fact_order_products fop
JOIN
  dim_aisles a ON fop.aisle_id = a.aisle_id
WHERE
  fop.reordered = TRUE
GROUP BY
  a.aisle
ORDER BY
  total_reordered DESC
LIMIT 5;
```

#### Average Number of Products Added to the Cart per Order by Day of the Week
```
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;
```
#### Top 10 Users with Highest Number of Unique Products Ordered
```
SELECT
  u.user_id,
  COUNT(DISTINCT fop.product_id) AS unique_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_users u ON fop.user_id = u.user_id
GROUP BY
  u.user_id
ORDER BY
  unique_products_ordered DESC
LIMIT 10;
```

## Conclusion
This project showcases the complete process of building a data warehouse in Snowflake, from data storage in an S3 bucket to loading data, building dimensional models, and performing analytics. By following these steps, you can gain insights into customer behavior and product performance using the Instacart Market Basket Analysis dataset.


