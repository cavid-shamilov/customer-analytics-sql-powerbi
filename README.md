# Customer Analytics — SQL Server & Power BI

## 1. Project Overview

This project focuses on customer analytics using the Sample Superstore dataset. The objective is to understand customer value, purchasing behavior, profitability, and retention-related patterns through SQL Server and Power BI.

The main business question is:

**Which customers are most valuable, what drives their value, and how can the business improve customer retention and profitability?**

The project will cover:

- Customer Segmentation
- New vs. Returning Customers
- Customer Value Analysis
- Customer Profitability
- Retention and Churn Risk
- RFM Analysis
- Business Insights and Recommendations

**Important:** The dataset does not contain an explicit churn status. Churn-related analysis will therefore use inactivity-based churn risk indicators rather than identifying confirmed churned customers.

## 2. Dataset Source

Dataset: [Sample Superstore Dataset — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)

**Original dataset:**
- Total records: 9,994
- Total columns: 21
- Country: United States
- Time period: January 2014 – December 2017
- Unique customers: 793
- Unique orders: 5,009

**Cleaned analytical dataset:**
- Final records: 9,342
- Total columns: 21
- Source table: `dbo.superstore2`
- Cleaned table: `dbo.superstore_clean`

## 3. Data Structure and Grain

One row represents an individual product line within an order.

An order may contain multiple product lines. Therefore, `Order_ID` alone does not uniquely identify a row.

`Row_ID` is unique across the original dataset and serves as the source row-level identifier.

The combination `Order_ID + Product_ID` is not a reliable unique key because the same product can appear in multiple lines of the same order.

## 4. Column Descriptions

| Column | Description |
|---|---|
| Row ID | Unique identifier for each dataset row |
| Order ID | Identifier for an order |
| Order Date | Date when the order was placed |
| Ship Date | Date when the order was shipped |
| Ship Mode | Shipping method selected for the order |
| Customer ID | Customer identifier |
| Customer Name | Customer's name |
| Segment | Customer segment: Consumer, Corporate, or Home Office |
| Country | Country where the order was placed |
| City | Customer's city |
| State | Customer's state |
| Postal Code | Geographic postal code |
| Region | Geographic sales region |
| Product ID | Product identifier |
| Category | Main product category |
| Sub-Category | Product subcategory |
| Product Name | Product name |
| Sales | Sales amount for the individual product line |
| Quantity | Number of units sold in the product line |
| Discount | Discount rate applied to the product line |
| Profit | Profit or loss associated with the product line |

**Metric notes:**
- Sales represents the product-line sales amount after the applied discount.
- Discount values are stored as decimal rates. For example, `0.20` represents a 20% discount.
- Profit can be negative, indicating a loss on a product line.
- Negative Profit values are retained because they represent relevant business outcomes.

## 5. Tools & Technologies

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- Power BI
- GitHub

## 6. Data Audit & Quality Assessment

A comprehensive data audit was performed before cleaning and analysis.

### A. Dataset Structure

| Check | Result |
|---|---:|
| Original rows | 9,994 |
| Columns | 21 |
| Row identifier | Row_ID |
| Raw staging structure | NVARCHAR columns |

The original dataset was imported into SQL Server as a raw staging table.

### B. Missing Values

All 21 columns were checked for NULL and blank values.

| Check | Result |
|---|---:|
| NULL values | 0 |
| Blank values | 0 |

No missing or blank values were detected.

### C. Duplicates & Identifiers

- `Row_ID` was unique across all 9,994 original records.
- Eight `Order_ID + Product_ID` combinations appeared more than once.
- Seven combinations represented different order lines.
- One combination contained two identical business records, differing only by `Row_ID`.

The exact duplicate records were:

- Row IDs: 3406 and 3407
- Order ID: US-2014-150119
- Product ID: FUR-CH-10002965
- Sales: 281.372
- Quantity: 2
- Discount: 0.30
- Profit: -12.0588

One copy was removed during cleaning.

### D. Data Types & Conversion

All tested values were successfully convertible to their intended data types.

No invalid conversions were detected for:
- Row_ID
- Order_Date
- Ship_Date
- Sales
- Quantity
- Discount
- Profit

The cleaned table now uses appropriate SQL Server data types.

### E. Date Validation

| Metric | Result |
|---|---|
| First Order Date | 2014-01-03 |
| Last Order Date | 2017-12-30 |
| First Ship Date | 2014-01-07 |
| Last Ship Date | 2018-01-05 |
| Ship dates before order dates | 0 |
| Future order dates | 0 |
| Future ship dates | 0 |

**Yearly Order Distribution — Original Dataset**

| Year | Order Lines | Unique Orders |
|---|---:|---:|
| 2014 | 1,993 | 969 |
| 2015 | 2,102 | 1,038 |
| 2016 | 2,587 | 1,315 |
| 2017 | 3,312 | 1,687 |
| **Total** | **9,994** | **5,009** |

Unique order volume increased from 969 in 2014 to 1,687 in 2017, approximately 74.1%. This is a volume observation and does not by itself establish improved business performance.

### F. Numeric Validation

**Original Dataset — Numeric Summary**

| Metric | Minimum | Maximum | Average |
|---|---:|---:|---:|
| Sales | 0.4440 | 22,638.4800 | 229.8580 |
| Quantity | 1 | 14 | 3.7896 |
| Discount | 0.00 | 0.80 | 0.1562 |
| Profit | -6,599.9780 | 8,399.9760 | 28.6569 |

**Profit Distribution**

| Profit Type | Rows |
|---|---:|
| Negative Profit | 1,871 |
| Zero Profit | 65 |
| Positive Profit | 8,058 |
| **Total** | **9,994** |

Approximately 18.73% of original product-line records had negative Profit.

All valid negative Profit values were preserved during cleaning.

All discount values were within the range of 0 to 0.80.

### G. Categorical Consistency

| Column | Unique Values |
|---|---:|
| Ship Mode | 4 |
| Segment | 3 |
| Region | 4 |
| Category | 3 |
| Sub-Category | 17 |
| Country | 1 |

No leading or trailing whitespace was detected in the six tested categorical columns.

### H. Customer Validation

- Unique Customer IDs: 793
- Unique Customer Names: 793
- No Customer ID was associated with multiple customer names.
- No Customer Name was associated with multiple Customer IDs.

Customer IDs and names had a consistent one-to-one mapping in the original dataset.

### I. Order Validation

Each Order ID was checked for consistency across:
- Customer ID
- Order Date
- Ship Date
- Ship Mode

No conflicting order-level attributes were detected.

Original unique orders: 5,009.

### J. Product Validation

The original dataset contained product mapping inconsistencies.

- Unique Product IDs: 1,862
- Unique Product Names: 1,850
- 32 Product IDs were associated with multiple product names.
- Multiple product names were associated with more than one Product ID.

Examples included:

| Product Name | Number of Product IDs |
|---|---:|
| Staples | 10 |
| Staple envelope | 9 |
| Easy-staple paper | 8 |
| Staples in misc. colors | 7 |
| Staple remover | 3 |
| Staple holder | 3 |
| Staple magnet | 2 |

Because neither Product ID nor Product Name provided a consistent mapping, records associated with these inconsistent product mappings were excluded from the cleaned analytical dataset.

### K. Geographic Validation

No State-to-Region or City-and-State-to-Region inconsistencies were identified.

Original Postal Code results:

| Metric | Result |
|---|---:|
| Unique Postal Codes | 631 |
| Minimum length | 4 |
| Maximum length | 5 |
| Non-numeric values | 0 |

Some northeastern US ZIP Codes had missing leading zeros.

Examples:

| Original Value | Cleaned Value |
|---|---|
| 6010 | 06010 |
| 1040 | 01040 |
| 3060 | 03060 |
| 7002 | 07002 |
| 2886 | 02886 |

Postal Codes were converted to VARCHAR(5), and leading zeros were restored.

### L. Business Logic Validation

No non-positive Sales or Quantity values were detected.

**Discount vs. Profit**

| Discount | Total Rows | Loss Rows | Loss Rate |
|---:|---:|---:|---:|
| 0.00 | 4,798 | 0 | 0.00% |
| 0.10 | 94 | 4 | 4.26% |
| 0.15 | 52 | 17 | 32.69% |
| 0.20 | 3,657 | 502 | 13.73% |
| 0.30 | 227 | 208 | 91.63% |
| 0.32 | 27 | 27 | 100.00% |
| 0.40 | 206 | 180 | 87.38% |
| 0.45 | 11 | 11 | 100.00% |
| 0.50 | 66 | 66 | 100.00% |
| 0.60 | 138 | 138 | 100.00% |
| 0.70 | 418 | 418 | 100.00% |
| 0.80 | 300 | 300 | 100.00% |

Higher discount levels were strongly associated with a higher proportion of loss-making product-line records.

This is an association, not proof of causation. Product mix, costs, and other business factors may also influence profitability.

**Shipping Duration**

| Metric | Result |
|---|---:|
| Minimum duration | 0 days |
| Maximum duration | 7 days |
| Average duration | 3.96 days |

### M. Outlier Analysis

The Interquartile Range (IQR) method was used to identify potential outliers.

| Column | Q1 | Q3 | IQR | Lower Bound | Upper Bound | Outliers |
|---|---:|---:|---:|---:|---:|---:|
| Sales | 17.28 | 209.94 | 192.66 | -271.71 | 498.93 | 1,167 |
| Profit | 1.72875 | 29.364 | 27.63525 | -39.724125 | 70.816875 | 1,881 |
| Quantity | 2 | 5 | 3 | -2.5 | 9.5 | 170 |

Quantity outlier distribution:

| Quantity | Rows |
|---:|---:|
| 10 | 57 |
| 11 | 34 |
| 12 | 23 |
| 13 | 27 |
| 14 | 29 |
| **Total** | **170** |

Outliers were preserved because they may represent legitimate business activity rather than data errors.

## 7. Data Cleaning

The original raw table was preserved without modification.

**Source table:** `dbo.superstore2`

**Cleaned table:** `dbo.superstore_clean`

The following cleaning operations were performed:

### 7.1. Data Type Conversion

All 21 columns were assigned appropriate SQL Server data types, including:
- INT for Row_ID and Quantity
- DATE for Order_Date and Ship_Date
- DECIMAL for Sales, Discount, and Profit
- VARCHAR and NVARCHAR for identifiers and descriptive fields

### 7.2. Postal Code Correction

Postal Codes were converted to VARCHAR(5).

Four-digit ZIP Codes were padded with leading zeros to preserve their correct five-digit format.

### 7.3. Product Mapping Inconsistencies

Records associated with inconsistent Product ID and Product Name mappings were removed.

- Inconsistent product pairs identified: 124
- Related sales records removed: 651

This decision was made to avoid unreliable product-level analysis.

### 7.4. Exact Duplicate Removal

One exact duplicate business record was removed.

The record with the smaller Row_ID was retained.

### 7.5. Preservation Rules

The following data was retained:
- Negative Profit values
- High Sales values
- High positive and negative Profit values
- Quantity values from 10 to 14
- Original discount rates

No valid numeric outliers were automatically removed.

## 8. Cleaning Results

| Metric | Result |
|---|---:|
| Original Rows | 9,994 |
| Rows after data type conversion | 9,994 |
| Inconsistent Product Pairs | 124 |
| Rows removed due to Product Inconsistencies | 651 |
| Exact Duplicate Rows Removed | 1 |
| **Final Cleaned Rows** | **9,342** |

A total of 652 rows were removed.

Approximately 93.48% of the original records were retained.

## 9. Post-Cleaning Validation

The cleaned dataset was validated after all cleaning operations.

| Validation Check | Result |
|---|---:|
| Cleaned row count | 9,342 |
| Data types | Correct |
| NULL values | 0 |
| Blank values | 0 |
| Row_ID duplicates | 0 |
| Product ID → multiple names | 0 |
| Product Name → multiple IDs | 0 |
| Invalid Postal Code length | 0 |
| Non-numeric Postal Codes | 0 |
| Ship dates before order dates | 0 |
| Non-positive Sales | 0 |
| Non-positive Quantity | 0 |
| Invalid Discount values | 0 |
| Exact duplicate business records | 0 |

**Conclusion:** The cleaned table passed all performed post-cleaning validation checks and is ready for customer analytics.

## 10. Project Progress

| Stage | Status |
|---|---|
| Dataset selection | Completed |
| SQL Server import | Completed |
| Data structure audit | Completed |
| Missing value audit | Completed |
| Duplicate and identifier audit | Completed |
| Data type validation | Completed |
| Date validation | Completed |
| Numeric validation | Completed |
| Categorical validation | Completed |
| Customer validation | Completed |
| Order validation | Completed |
| Product validation | Completed |
| Geographic validation | Completed |
| Business logic validation | Completed |
| Outlier analysis | Completed |
| Data cleaning | Completed |
| Post-cleaning validation | Completed |
| Customer analysis | Pending |
| Customer segmentation | Pending |
| Customer profitability | Pending |
| Retention and churn risk | Pending |
| RFM analysis | Pending |
| Power BI dashboard | Pending |
| Business findings and recommendations | Pending |

## 11. Next Steps

The project will now move into the analytical phase:

1. Customer-level sales and profit analysis.
2. New vs. Returning Customer Analysis.
3. Customer Value and Purchase Frequency.
4. Customer Profitability Analysis.
5. Customer Segmentation.
6. Retention and Inactivity-Based Churn Risk.
7. RFM Analysis.
8. Power BI Dashboard Development.
9. Business Findings and Recommendations.

---

*This project is part of a practical portfolio focused on SQL-based business analysis, customer insights, and data-driven decision-making.*

Task 1 — Customer Overview

How many unique customers are present in the cleaned dataset? Also calculate the total number of order lines and unique orders.

```sql
select count(distinct Customer_ID) as [Number of Unique Customers] ,
count(order_id) as [Number of Order Lines],
count(distinct order_id) as [Number of Unique Orders]
from superstore_clean
```

Task 2 — Customer Sales

Calculate the total Sales generated by each customer. Display Customer_ID, Customer_Name, and Total_Sales. Sort by Total_Sales in descending order.

```sql
select customer_id,
customer_name,
sum(sales) as [Total Sales]
from superstore_clean
group by customer_id,customer_name
order by [Total Sales] desc
```

Task 3 — Customer Profit

Calculate the total Profit generated by each customer. Display Customer_ID, Customer_Name, and Total_Profit. Sort by Total_Profit in descending order.

```sql
select customer_id,
customer_name,
sum(Profit) as [Total Profit]
from superstore_clean
group by customer_id,customer_name
order by [Total Profit] desc
```

Task 4 — Customer Orders

Calculate the number of unique orders placed by each customer. Display Customer_ID, Customer_Name, and Order_Count.

```sql
select customer_id,
customer_name,
count(distinct Order_ID) as [Unique Orders]
from superstore_clean
group by customer_id,customer_name
order by [Unique Orders] desc
```

Task 5 — Purchase Frequency

Calculate the average number of product lines per order for each customer. Display Customer_ID, Customer_Name, Total_Order_Lines, Order_Count, and Lines_Per_Order.

```sql
select
Customer_ID,
Customer_Name,
count(*) as Total_Order_Lines,
count(distinct Order_ID) as Order_Count,
cast(count(*) as decimal(10,2)) /
count(distinct Order_ID) as Lines_Per_Order
from superstore_clean
group by
Customer_ID,
Customer_Name
order by Lines_Per_Order desc
```

Task 6 — Average Order Value (AOV)

Calculate each customer's Average Order Value using total Sales divided by the number of unique orders. Display Customer_ID, Customer_Name, Total_Sales, Order_Count, and AOV.

```sql
select  Customer_ID,Customer_Name,
sum(sales) as [Total Sales],
count(distinct order_id) as [Order_Count],
sum(sales)/count(distinct order_id) as AOV
from superstore_clean
group by Customer_ID,Customer_Name
order by AOV desc
```

Task 7 — Average Profit per Order

Calculate the average Profit per unique order for each customer. Display Customer_ID, Customer_Name, Total_Profit, Order_Count, and Average_Profit_Per_Order.

```sql
select  Customer_ID,Customer_Name,
sum(Profit) as [Profit],
count(distinct order_id) as [Order_Count],
sum(Profit)/count(distinct order_id) as Average_Profit_Per_Order
from superstore_clean
group by Customer_ID,Customer_Name
order by Average_Profit_Per_Order desc
```
Task 8 — Customer Quantity

Calculate the total quantity of products purchased by each customer. Display Customer_ID, Customer_Name, and Total_Quantity. Sort in descending order.

```sql
select  Customer_ID,Customer_Name,
sum(quantity) as [Total Quantity]
from superstore_clean
group by Customer_ID,Customer_Name
order by [Total Quantity] desc
```
Task 9 — Customer Discount

Calculate the average discount rate received by each customer. Display Customer_ID, Customer_Name, and Average_Discount. Sort from highest to lowest.

```sql
select  Customer_ID,Customer_Name,
avg(Discount) as Average_Discount
from superstore_clean
group by Customer_ID,Customer_Name
order by Average_Discount desc
```

Task 10 — Customer Performance Summary

Create a customer-level summary containing Total_Sales, Total_Profit, Order_Count, Total_Quantity, AOV, and Average_Profit_Per_Order. Sort customers by Total_Sales in descending order.

```sql
select  Customer_ID,Customer_Name,
sum(sales) as [Total Sales],
sum(profit) as [Total Profit],
count(distinct order_id) as [Order Count],
sum(quantity) as [Total Quantity],
sum(sales)/count(distinct order_id) as AOV,
sum(Profit)/count(distinct order_id) as Average_Profit_Per_Order
from superstore_clean
group by Customer_ID,Customer_Name
order by [Total Sales] desc
```

Task 11 — First Purchase Date
Find the first purchase date for each customer. Display Customer_ID, Customer_Name, and First_Purchase_Date. Sort by First_Purchase_Date ascending.

```sql
select customer_id,
Customer_Name,
min(order_date) as First_Purchase_Date 
from superstore_clean
group by customer_id,
Customer_Name
order by First_Purchase_Date  
```
Task 12 — Last Purchase Date
Find the most recent purchase date for each customer. Display Customer_ID, Customer_Name, and Last_Purchase_Date. Sort by Last_Purchase_Date descending.

```sql
select customer_id,
Customer_Name,
max(order_date) as Last_Purchase_Date 
from superstore_clean
group by customer_id,
Customer_Name
order by Last_Purchase_Date desc  
```
Task 13 — Customer Purchase Lifespan
Calculate the number of days between each customer's first and last purchase. Display Customer_ID, Customer_Name, First_Purchase_Date, Last_Purchase_Date, and Customer_Lifespan_Days. Sort by Customer_Lifespan_Days descending.

```sql
select customer_id,
Customer_Name,
min(order_date) as First_Purchase_Date,
max(order_date) as Last_Purchase_Date ,
datediff(day,min(order_date),max(order_date)) as Customer_Lifespan_Days
from superstore_clean
group by customer_id,
Customer_Name
order by Customer_Lifespan_Days desc 
```

Task 14 — Customer Order History
Create an order-level summary for every customer. Display:
•	Customer_ID
•	Customer_Name
•	Order_ID
•	Order_Date
•	Order_Line_Count
•	Order_Sales
•	Order_Profit
•	Order_Quantity
Each customer-order combination should appear only once. Sort by Customer_ID and Order_Date.

```sql
select customer_id,
Customer_Name,
order_id,
order_date,
count(order_id) as Order_Line_Count,
sum(sales) as Order_Sales,
sum(profit) as Order_Profit,
sum(Quantity) as Order_Quantity	
from superstore_clean
group by customer_id,
Customer_Name,
order_id,
Order_Date
order by  Customer_ID,Order_Date
```

Task 15 — One-Time vs. Repeat Customers
Classify each customer into one of two groups:
•	One-Time Customer: placed exactly one unique order.
•	Repeat Customer: placed more than one unique order.
Display Customer_ID, Customer_Name, Order_Count, and Customer_Type.

```sql
select customer_id,
Customer_Name,
count( distinct Order_ID) as Order_Count,
case 
when(count(distinct Order_ID))=1 then 'One-Time Customer'
else 'Repeat Customer'
end as Customer_Type
from superstore_clean
group by customer_id,
Customer_Name
```

Task 16 — One-Time vs. Repeat Customer Distribution
Calculate the number of One-Time Customers and Repeat Customers. Also calculate their percentage of total customers.
Display Customer_Type, Customer_Count, and Customer_Percentage.

```sql
with customer_type_table as (
select
Customer_ID,
case
when count(distinct Order_ID) = 1 then 'One-Time Customer'
else 'Repeat Customer'
end as Customer_Type
from superstore_clean
group by Customer_ID
)
select 
Customer_Type,
count(*) as Customer_Count,
cast(count(*) * 100.0 / sum(count(*)) OVER () as decimal(10, 2)) as Customer_Percentage
from customer_type_table
group by Customer_Type
```

Task 17 — New Customers by Year
Calculate the number of new customers acquired in each year.
A customer is considered new in the year of their first-ever purchase.
Display First_Purchase_Year and New_Customer_Count.

```sql
with First_Purchase_Years as(
select customer_id,
min(year(order_date)) as First_Purchase_Year
from superstore_clean
group by customer_id
)
select First_Purchase_Year,
count(customer_id) as New_Customer_Count
from First_Purchase_Years
group by First_Purchase_Year
order by First_Purchase_Year
```

Task 18 — New vs. Returning Customers by Year
For each calendar year, calculate:
•	Total_Active_Customers
•	New_Customers
•	Returning_Customers
A returning customer is someone who made a purchase during that year but whose first purchase occurred in an earlier year.

```sql
WITH FirstPurchaseYear AS (
    SELECT 
        Customer_ID,
        YEAR(MIN(Order_Date)) AS First_Purchase_Year
    FROM superstore_clean
    GROUP BY Customer_ID
),
ActiveCustomersByYear AS (
    SELECT DISTINCT
        Customer_ID,
        YEAR(Order_Date) AS Calendar_Year
    FROM superstore_clean
)
SELECT 
    a.Calendar_Year,
    COUNT(a.Customer_ID) AS Total_Active_Customers,
    COUNT(CASE WHEN f.First_Purchase_Year = a.Calendar_Year THEN a.Customer_ID END) AS New_Customers,
    COUNT(CASE WHEN f.First_Purchase_Year < a.Calendar_Year THEN a.Customer_ID END) AS Returning_Customers
FROM ActiveCustomersByYear a
JOIN FirstPurchaseYear f
    ON a.Customer_ID = f.Customer_ID
GROUP BY a.Calendar_Year
ORDER BY a.Calendar_Year;
```

