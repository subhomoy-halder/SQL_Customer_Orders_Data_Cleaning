# Customer Orders Data leaning Pipeline | SQL

## Project Objective
In industry, raw data is rarely ready for immediate analysis. This project demonstrates a comprehensive data cleaning pipeline built to transform messy, inconsistent and raw ecommerce sales data into a structured production ready format. 

The goal of this project is to identify and resolve common data quality issues such as formatting inconsistencies, missing values and duplicate records ensuring the final dataset is reliable for business intelligence and reporting.

## Tools & Techniques Used
* **Database Management System:** Microsoft SQL Server (T-SQL)
* **Techniques:** Common Table Expressions (CTEs), Window Functions, String Manipulation, Data Type Conversion, DML/DDL Commands.

---

## Code Highlights

### 1. Removing Duplicates using a CTE and Window Functions
To ensure revenue wasn't double counted, I partitioned the data by all relevant columns to isolate and delete exact duplicates while keeping the original record.

```sql
WITH duplicates AS (
    SELECT
        order_id,
        ROW_NUMBER() OVER(
            PARTITION BY customer_name, email, order_date, product_name, quantity, price, country, order_status 
            ORDER BY order_date
        ) AS rn
    FROM data_cleaning_2_cleaned
)
DELETE FROM data_cleaning_2_cleaned
WHERE order_id IN (
    SELECT order_id 
    FROM duplicates
    WHERE rn > 1
);
```

### 2. Standardizing Categorical Data
To prevent aggregation errors in BI tools, I used string manipulation and `CASE WHEN` statements to funnel inconsistent entries into clean categories.

```sql
UPDATE data_cleaning_2_cleaned
SET country = CASE
                WHEN LOWER(country) IN ('uk', 'united kingdom') THEN 'United Kingdom'
                WHEN LOWER(country) IN ('us', 'usa', 'united states', 'united states of america') THEN 'United States of America'
                WHEN LOWER(country) = 'canada' THEN 'Canada'
                WHEN LOWER(country) = 'spain' THEN 'Spain'
                WHEN LOWER(country) = 'india' THEN 'India'
                ELSE 'Others'
              END;
```

---

## The Result: Before & After

Here is a snapshot of how the raw data was transformed into analysis ready data:

| Aspect | Raw Data (Before) | Cleaned Data (After) |
| :--- | :--- | :--- |
| **Inconsistent Case & Formatting** | `1008, Carlos Hern+índez, Iphone 14, spain, "DELIVERED,-"` | `1008, Carlos Hernandez, iPhone 14, Spain, Delivered` |
| **Data Type Typos** | `1003, SARAH THOMPSON, Samsung Galaxy S22, two, 799` | `1003, Sarah Thompson, Samsung Galaxy S22, 2, 799.00` |
| **Missing Values** | `1004, Tom O'Brien, NULL, Google Pixel` | `1004, Tom O'Brien, tom.obrien@gmail.com, Google Pixel` |
| **Date Formatting** | `1002, john smith, 11/02/2023, apple watch` | `1002, John Smith, 2023-11-02, Apple Watch` |

---

## The 14 Step Data Cleaning Process
During the data exploration phase, 14 specific data discrepancies were identified. The `Data Cleaning Scripts.sql` file addresses each of them in the following order:

1. **Standardizing Customer Names:** Utilized string manipulation (`CHARINDEX`, `UPPER`, `LOWER`, `SUBSTRING`, `TRIM`) to dynamically format all names to proper Title Case.
2. **Standardizing Order Dates:** Used the `CAST` function to standardize the column into a proper SQL `DATE` format (`YYYY-MM-DD`).
3. **Standardizing Product Names:** Applied a `CASE WHEN` statement combined with `LOWER()` to categorize products into their official brand formatting.
4. **Fixing Alphabetical Quantities:** Updated text strings (e.g., `two`) in numerical columns to their corresponding integers.
5. **Cleaning Financial Formatting:** Used `RIGHT` and `LEN` functions to strip leading non numeric characters (like `$`) to prepare the column for math operations.
6. **Standardizing Country Names:** Used a `CASE WHEN` statement to map regional variations (`uk`, `USA`) to standardized names.
7. **Standardizing Order Statuses:** Normalized statuses to Title Case using a `CASE WHEN` statement.
8. **Handling Missing Names:** Replaced literal `'NULL'` strings and actual `NULL` values in the `customer_name` column with `'Unknown'`.
9. **Handling Missing Emails:** Utilized existing data matching to impute missing email addresses where applicable.
10. **Imputing Missing Prices:** Built a lookup `CASE` statement based on `product_name` to backfill missing prices with correct retail values.
11. **Removing Invalid Characters:** Used pattern matching (`LIKE '%[^A-Za-z ]%'`) to identify and correct names with corrupted symbols.
12. **Removing Duplicate Records:** Created a CTE utilizing the `ROW_NUMBER()` Window Function to safely delete exact row level duplicates.
13. **Casting Data Types:** Used `ALTER TABLE` and `ALTER COLUMN` to convert strings into optimal structural data types (`INT`, `DATE`, `DECIMAL`).
14. **Dropping Unnecessary Columns:** Dropped the `notes` column to reduce table bloat.

---
