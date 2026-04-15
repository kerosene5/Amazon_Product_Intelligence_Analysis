# Overview

![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat&logo=mysql&logoColor=white)
![Dataset](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?style=flat&logo=kaggle&logoColor=white)
![Records](https://img.shields.io/badge/Records-1%2C465-brightgreen?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat)

> A structured SQL-based analysis of 1,337 Amazon India products where I explore pricing, discounts, ratings, and category performance using EDA and Advanced Analytics techniques.

## Objective

The main goal is to:
- Understand the structure and quality of real-world data
- Perform meaningful analysis using SQL
- Derive actionable insights related to products, pricing, and customer behavior

---

This project aims to answer business questions about Amazon's product catalog:
- Which product categories dominate the platform?
- What is the relationship between discount percentage and customer rating?
- Which products and categories deliver the best value?
- How are products distributed across price and discount segments?
- What does the top-performing vs. bottom-performing product landscape look like?

## Dataset

**Source:** [Amazon Sales Dataset (from Kaggle)](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset/data)

This dataset contains 1,000+ Amazon product listings with ratings and reviews as per product information listed on [Amazon India](https://www.amazon.in/)

| Column | Description |
|---|---|
| `product_id` | Unique product identifier |
| `product_name` | Name of the product |
| `category` | Hierarchical category path |
| `discounted_price` | Selling price (₹) |
| `actual_price` | MRP / original price (₹) |
| `discount_percentage` | Discount offered (%) |
| `rating` | Average customer rating (out of 5) |
| `rating_count` | Number of ratings received |
| `about_product` | Product description |
| `user_id` / `user_name` | Reviewer identifiers |
| `review_id` / `review_title` / `review_content` | Review details |

---

## Framework

The analysis follows this structured mind map:

![Analysis Framework](https://github.com/user-attachments/assets/8008fb3b-d496-4e1f-9539-261bcf3837b4)

---

## Data Cleaning

The raw CSV file requires encoding fixes (`₹` symbol causes `charmap` codec errors in MySQL Workbench)

```python
import pandas as pd
df = pd.read_csv('amazon.csv', encoding='latin1')
for col in df.select_dtypes(include='object').columns:
    df[col] = df[col].astype(str).str.encode('ascii', errors='ignore').str.decode('ascii')
df.to_csv('amazon_clean.csv', index=False, encoding='utf-8')
```

After importing `amazon_clean.csv` in MySQL Workbench, numeric columns were extracted from raw string fields (for example, `₹1,099` is changed to `1099.00`):

```sql
UPDATE products SET
    discounted_price_num = CAST(NULLIF(REPLACE(REPLACE(discounted_price, '?', ''), ',', ''), '') AS DECIMAL(10,2)),
    actual_price_num     = CAST(NULLIF(REPLACE(REPLACE(actual_price, '?', ''), ',', ''), '') AS DECIMAL(10,2)),
    discount_pct_num     = CAST(NULLIF(REPLACE(discount_percentage, '%', ''), '') AS DECIMAL(5,2)),
    rating_num           = CAST(NULLIF(REPLACE(REPLACE(rating, '|', ''), ',', ''), '') AS DECIMAL(3,1)),
    rating_count_num     = CAST(NULLIF(TRIM(REPLACE(rating_count, ',', '')), '') AS UNSIGNED);
```

Now the dataset is ready for analysis!

---

# Exploratory Data Analysis

Let us proceed with the EDA of the dataset to get a thorough feel and familiarize ourselves with the data!

## Database Exploration

So, since this is a completely new dataset and we have no idea what we're in for, 
we first perform Database Exploration to understand what tables there are in the database (in this case, there is only 1 table).

 ```sql
SELECT * FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'amazon_sales';
```

 ```sql
SELECT * FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'products';
```
