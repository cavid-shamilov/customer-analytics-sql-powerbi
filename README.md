# Customer Analytics — SQL Server & Power BI

## 1. Project Overview

This project focuses on customer analytics using the Sample Superstore dataset. The objective is to understand customer value, purchasing behavior, profitability, and retention-related patterns through SQL Server and Power BI.

The project aims to answer the following business question:

**Which customers are most valuable, what drives their value, and how can the business improve customer retention and profitability?**

The analysis will cover:

- Customer Segmentation
- New vs. Returning Customers
- Customer Value Analysis
- Customer Profitability
- Retention and Churn Risk
- RFM Analysis
- Business Insights and Recommendations

**Important:** The dataset does not contain an explicit churn status. Churn-related analysis will therefore use inactivity-based churn risk indicators rather than claiming to identify confirmed churned customers.

## 2. Dataset Source

Dataset: [Sample Superstore Dataset — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)

- Total records: 9,994
- Total columns: 21
- Country: United States
- Time period: January 2014 – December 2017
- Unique customers: 793
- Unique orders: 5,009

## 3. Data Structure and Grain

The dataset contains order, customer, product, shipping, geographic, sales, and profit information.

**Data Grain:** One row represents an individual product line within an order.

An order may contain multiple product lines. Therefore, `Order_ID` alone does not uniquely identify a row.

`Row_ID` is unique across all 9,994 records and serves as the row-level identifier in the source dataset.

The combination `Order_ID + Product_ID` is not a reliable unique key because the same product can appear in multiple lines of the same order.

## 4. Column Descriptions

| Column | Description |
|---|---|
| Row ID | Unique identifier for each dataset row |
| Order ID | Unique identifier for an order |
| Order Date | Date when the order was placed |
| Ship Date | Date when the order was shipped |
| Ship Mode | Shipping method selected for the order |
| Customer ID | Unique customer identifier |
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
- Sales values represent individual product-line sales amounts after the applied discount.
- Discount values are stored as decimal rates. For example, `0.20` represents a 20% discount.
- Profit can be negative, indicating a loss on a product line.
- Negative Profit values are retained because they represent relevant business outcomes.

## 5. Tools & Technologies

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- Power BI
- GitHub

## 6. Data Audit & Quality Assessment

A comprehensive data audit was performed before data cleaning and analysis.

### A. Dataset Structure

| Check | Result | Status |
|---|---:|---|
| Total rows | 9,994 | Passed |
| Total columns | 21 | Passed |
| Row identifier | Row_ID | Passed |
| Raw column types | NVARCHAR staging structure | Expected |

The dataset was successfully imported into SQL Server as a raw staging table.

### B. Missing Values

All 21 columns were checked for NULL and blank values.

| Check | Result |
|---|---:|
| Total rows | 9,994 |
| NULL values | 0 |
| Blank values | 0 |

**Conclusion:** No missing or blank values were detected in the dataset.

### C. Duplicates & Identifiers

- `Row_ID` is unique across all 9,994 records.
- Eight `Order_ID + Product_ID` combinations appeared more than once.
- Seven combinations represented different order lines, with differences in quantity and other sales values.
- One combination contained two identical business records, differing only in `Row_ID`.

The two matching records were:

- Row IDs: 3406 and 3407
- Order ID: US-2014-150119
- Product ID: FUR-CH-10002965
- Sales: 281.372
- Quantity: 2
- Discount: 0.30
- Profit: -12.0588

**Conclusion:** One potential exact duplicate business record was identified. It will be investigated before deciding whether to exclude one copy from the cleaned dataset. The raw table will remain unchanged.

### D. Data Types & Conversion

All relevant fields were tested using SQL Server `TRY_CONVERT()`.

| Field | Invalid Conversions |
|---|---:|
| Row_ID | 0 |
| Order_Date | 0 |
| Ship_Date | 0 |
| Sales | 0 |
| Quantity | 0 |
| Discount | 0 |
| Profit | 0 |

**Conclusion:** All tested values can be converted to their intended data types.

The raw table uses NVARCHAR columns intentionally as a staging structure. Appropriate data types will be applied in a separate cleaned table.

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

**Yearly Order Distribution**

| Year | Order Lines | Unique Orders |
|---|---:|---:|
| 2014 | 1,993 | 969 |
| 2015 | 2,102 | 1,038 |
| 2016 | 2,587 | 1,315 |
| 2017 | 3,312 | 1,687 |
| **Total** | **9,994** | **5,009** |

Unique order volume increased from 969 in 2014 to 1,687 in 2017, approximately 74.1%.

This is a preliminary volume observation and does not by itself establish improved business performance or profitability.

### F. Numeric Validation

**Numeric Summary**

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

Approximately 18.73% of product-line records have negative Profit.

Negative Profit values are valid business outcomes and will not be removed.

**Discount Distribution**

| Discount | Rows |
|---:|---:|
| 0.00 | 4,798 |
| 0.10 | 94 |
| 0.15 | 52 |
| 0.20 | 3,657 |
| 0.30 | 227 |
| 0.32 | 27 |
| 0.40 | 206 |
| 0.45 | 11 |
| 0.50 | 66 |
| 0.60 | 138 |
| 0.70 | 418 |
| 0.80 | 300 |

All discount values are within the valid range of 0 to 0.80.

### G. Categorical Consistency

The following distinct category values were identified:

| Column | Unique Values |
|---|---:|
| Ship Mode | 4 |
| Segment | 3 |
| Region | 4 |
| Category | 3 |
| Sub-Category | 17 |
| Country | 1 |

No leading or trailing whitespace was detected in the six tested categorical columns.

**Conclusion:** No categorical whitespace inconsistencies were identified in the tested fields.

### H. Customer Validation

- Unique Customer IDs: 793
- Unique Customer Names: 793
- No Customer ID was associated with multiple customer names.
- No Customer Name was associated with multiple Customer IDs.

**Conclusion:** Customer IDs and customer names have a consistent one-to-one mapping in this dataset.

### I. Order Validation

Each Order ID was checked for consistency across:

- Customer ID
- Order Date
- Ship Date
- Ship Mode

No orders were found with conflicting values across these attributes.

Total unique orders: 5,009.

**Conclusion:** Order-level attributes are consistent across the records.

### J. Product Validation

Product validation revealed an important data-quality issue.

- Unique Product IDs: 1,862
- Unique Product Names: 1,850
- Unique Categories: 3
- Unique Sub-Categories: 17

**Product ID to Product Name inconsistency:**

32 Product IDs were associated with two different product names each.

Further investigation showed that these were different product names rather than simple spelling variations. Their categories and subcategories remained consistent.

**Product Name to Product ID inconsistency:**

Several product names were associated with multiple Product IDs.

Examples include:

| Product Name | Number of Product IDs |
|---|---:|
| Staples | 10 |
| Staple envelope | 9 |
| Easy-staple paper | 8 |
| Staples in misc. colors | 7 |
| Staple remover | 3 |
| Staple holder | 3 |
| Staple magnet | 2 |
| Avery Non-Stick Binders | 2 |
| Eldon Wave Desk Accessories | 2 |

**Conclusion:** Neither Product ID nor Product Name can be treated as a unique, fully reliable product key without a deliberate mapping or standardization decision.

The original product identifiers and names will be preserved in the raw table. Product-level analysis will account for this limitation.

### K. Geographic Validation

Geographic consistency checks included:

- State-to-Region mapping
- City-and-State-to-Region mapping
- Postal Code format validation

No State-to-Region or City-and-State-to-Region inconsistencies were identified.

Postal Code results:

| Metric | Result |
|---|---:|
| Total rows | 9,994 |
| Unique Postal Codes | 631 |
| Minimum length | 4 |
| Maximum length | 5 |
| Non-numeric values | 0 |

Some Postal Codes contain only four digits. These occur in northeastern US states where leading zeros are expected.

Examples:

| Original Value | Expected Format |
|---|---|
| 6010 | 06010 |
| 1040 | 01040 |
| 3060 | 03060 |
| 7002 | 07002 |
| 2886 | 02886 |

**Conclusion:** Postal Codes should be stored as text, and missing leading zeros should be restored during cleaning.

### L. Business Logic Validation

**Sales and Quantity**

- No non-positive Sales values were found.
- No non-positive Quantity values were found.
- All tested Sales and Quantity values passed the conversion and validity checks.

**Discount vs. Profit**

The relationship between discount levels and loss records was investigated.

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

**Key observation:** Higher discount levels are strongly associated with a higher proportion of loss-making product-line records in this dataset.

For example:
- At a 30% discount, 91.63% of records had negative Profit.
- At a 40% discount, 87.38% had negative Profit.
- At discount levels of 45% or higher, all observed records had negative Profit.

This is an association, not proof of causation. Product mix, costs, and other business factors may also influence profitability.

This finding will be investigated further during the business analysis.

**Shipping Duration**

| Metric | Result |
|---|---:|
| Minimum shipping duration | 0 days |
| Maximum shipping duration | 7 days |
| Average shipping duration | 3.96 days |

No shipment was recorded before its corresponding order date.

### M. Outlier Analysis

The Interquartile Range (IQR) method was used to identify potential outliers in Sales, Profit, and Quantity.

The standard boundaries were calculated as:

- Lower Bound = Q1 − 1.5 × IQR
- Upper Bound = Q3 + 1.5 × IQR

**Outlier Summary**

| Column | Q1 | Q3 | IQR | Lower Bound | Upper Bound | Outliers |
|---|---:|---:|---:|---:|---:|---:|
| Sales | 17.28 | 209.94 | 192.66 | -271.71 | 498.93 | 1,167 |
| Profit | 1.72875 | 29.364 | 27.63525 | -39.724125 | 70.816875 | 1,881 |
| Quantity | 2 | 5 | 3 | -2.5 | 9.5 | 170 |

**Quantity Outlier Distribution**

| Quantity | Rows |
|---:|---:|
| 10 | 57 |
| 11 | 34 |
| 12 | 23 |
| 13 | 27 |
| 14 | 29 |
| **Total** | **170** |

**Conclusion:** Outliers were identified in all three numeric fields. These values are not automatically considered errors. High sales, unusually high profits, significant losses, and larger order quantities can represent legitimate business activity.

No outliers will be automatically deleted.

### N. Final Audit Summary

The dataset passed most structural, completeness, conversion, date, customer, order, and geographic checks.

However, several issues require deliberate cleaning decisions before analysis:

1. One potential exact duplicate business record.
2. Product ID and Product Name inconsistencies.
3. Four-digit Postal Codes that require leading-zero restoration.
4. Numeric outliers that require preservation and contextual interpretation.

**Overall conclusion:** The dataset is suitable for further analysis after applying documented cleaning rules and addressing the identified data-quality limitations.

## 7. Data Cleaning Strategy

The cleaning stage will be performed in a separate table, preserving the original raw data.

Planned actions:

- Convert dates into DATE data types.
- Convert numeric columns into appropriate numeric types.
- Preserve Postal Codes as text and restore leading zeros.
- Investigate the potential exact duplicate record before deciding whether to exclude one copy.
- Preserve original product identifiers and names; avoid arbitrary product consolidation.
- Retain valid negative Profit values.
- Retain valid numeric outliers.
- Document all cleaning decisions.

## 8. Project Progress

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
| Final audit summary | Completed |
| Data cleaning | Pending |
| Customer analysis | Pending |
| RFM analysis | Pending |
| Power BI dashboard | Pending |
| Business findings and recommendations | Pending |

## 9. Next Steps

1. Create a separate cleaned SQL table.
2. Apply the documented cleaning rules.
3. Validate the cleaned dataset.
4. Perform customer segmentation and customer value analysis.
5. Analyze customer profitability and purchasing behavior.
6. Investigate inactivity-based retention and churn risk.
7. Conduct RFM analysis.
8. Build an interactive Power BI dashboard.
9. Document final business insights and recommendations.

---

*This project is part of a practical portfolio focused on SQL-based business analysis and data-driven decision-making.*
