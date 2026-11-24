# Customer Analytics — SQL Server & Power BI

## Project Overview

This project focuses on customer analytics using the Sample Superstore dataset. The goal is to understand customer behavior, identify valuable customer groups, evaluate customer profitability, and explore opportunities to improve customer retention.

The main business question is:

**Which customers are most valuable, what drives their value, and how can the business improve customer retention and profitability?**

The analysis will cover:

- Customer Segmentation
- New vs. Returning Customers
- Customer Value and Purchase Frequency
- Customer Profitability
- Retention and Inactivity-Based Churn Risk
- RFM Analysis (Recency, Frequency, Monetary)
- Interactive Power BI Dashboard
- Business Insights and Recommendations

## Dataset Source

Dataset: [Sample Superstore Dataset — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)

- Total records: **9,994**
- Total columns: **21**
- Country coverage: **United States**
- Order date range: **January 3, 2014 – December 30, 2017**

## Data Grain

Each row represents an individual product line within an order.

An order can contain multiple product lines. Therefore, `Order_ID` alone does not uniquely identify a row. The dataset includes `Row_ID` as a row-level identifier.

The combination of `Order_ID` and `Product_ID` is not guaranteed to be unique, because some orders contain multiple lines with the same product ID.

## Dataset Columns

| Column | Description |
|---|---|
| Row ID | Unique identifier assigned to each row in the dataset. |
| Order ID | Identifier of the customer order. |
| Order Date | Date when the order was placed. |
| Ship Date | Date when the order was shipped. |
| Ship Mode | Shipping method selected for the order. |
| Customer ID | Unique customer identifier in the source dataset. |
| Customer Name | Customer's name. |
| Segment | Customer segment: Consumer, Corporate, or Home Office. |
| Country | Country where the order was placed. |
| City | City associated with the order. |
| State | State associated with the order. |
| Postal Code | Postal code associated with the order's location. |
| Region | Geographic sales region. |
| Product ID | Product identifier provided by the source dataset. |
| Category | Main product category. |
| Sub-Category | More specific product classification. |
| Product Name | Product name provided by the source dataset. |
| Sales | Sales amount for the individual order line, after the applied discount. |
| Quantity | Number of units sold in the order line. |
| Discount | Discount rate applied to the order line. |
| Profit | Profit or loss recorded for the individual order line. Negative values represent losses. |

## Tools & Technologies

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- Power BI
- GitHub

## Data Audit & Quality Assessment

The initial data audit was performed on the raw staging table:

`dbo.superstore2`

All source columns were imported as text (`NVARCHAR`) to preserve the original values and avoid incorrect numeric or date conversions during import.

The raw staging table has been kept unchanged.

### A. Dataset Structure

| Check | Result |
|---|---:|
| Total rows | 9,994 |
| Total columns | 21 |
| Raw table | `dbo.superstore2` |
| Initial column storage | NVARCHAR |

**Status:** Completed.

### B. Missing Values

All 21 columns were checked for SQL `NULL` values and empty strings.

| Check | Result |
|---|---:|
| Columns checked | 21 |
| Columns with NULL values | 0 |
| Columns with blank values | 0 |

No missing or blank values were detected in the raw dataset.

**Status:** Completed.

### C. Duplicates & Identifiers

- `Row_ID` was checked for duplicate values. No duplicates were found.
- A duplicate check using `Order_ID` and `Product_ID` identified 8 repeated combinations.
- Seven combinations represented separate order lines with different quantities and corresponding business values.
- One combination contained two identical business records, differing only by `Row_ID`.

The potential exact duplicate:

- Order ID: `US-2014-150119`
- Product ID: `FUR-CH-10002965`
- Row IDs: `3406` and `3407`
- Sales: `281.372`
- Quantity: `2`
- Discount: `0.30`
- Profit: `-12.0588`

This record will be investigated during the cleaning stage. It has not been deleted from the raw data.

**Status:** Completed.

### D. Data Types & Conversion

The following fields were tested for conversion into their expected data types:

- `Row_ID` → INT
- `Order_Date` → DATE
- `Ship_Date` → DATE
- `Sales` → DECIMAL
- `Quantity` → INT
- `Discount` → DECIMAL
- `Profit` → DECIMAL

All 9,994 records passed the conversion checks. No invalid values were detected in these fields.

**Status:** Completed.

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

Yearly order activity:

| Year | Order Lines | Unique Orders |
|---|---:|---:|
| 2014 | 1,993 | 969 |
| 2015 | 2,102 | 1,038 |
| 2016 | 2,587 | 1,315 |
| 2017 | 3,312 | 1,687 |
| **Total** | **9,994** | **5,009** |

The number of unique orders increased from 969 in 2014 to 1,687 in 2017. This is a preliminary volume observation and does not, by itself, establish an improvement in revenue or profitability.

**Status:** Completed.

### F. Numeric Validation

Summary of numeric fields:

| Metric | Minimum | Maximum | Average |
|---|---:|---:|---:|
| Sales | 0.4440 | 22,638.4800 | 229.8580 |
| Quantity | 1 | 14 | 3.7896 |
| Discount | 0.00 | 0.80 | 0.1562 |
| Profit | -6,599.9780 | 8,399.9760 | 28.6569 |

Business rule checks:

| Check | Result |
|---|---:|
| Non-positive Sales | 0 |
| Non-positive Quantity | 0 |
| Discounts outside 0–1 | 0 |
| Negative Profit rows | 1,871 |
| Zero Profit rows | 65 |
| Positive Profit rows | 8,058 |

Approximately **18.73%** of order lines have negative profit.

Negative profit values are valid business outcomes and should not be treated as data errors.

Discount values are already stored as decimal rates. For example, `0.20` represents a 20% discount. No additional division or multiplication is required to interpret the source values.

Negative profit should not automatically be attributed to discounts alone because the dataset does not provide a complete breakdown of all cost components.

**Status:** Completed.

### G. Categorical Consistency

The following distinct values were identified:

| Column | Distinct Values |
|---|---:|
| Ship Mode | 4 |
| Segment | 3 |
| Region | 4 |
| Category | 3 |
| Sub-Category | 17 |
| Country | 1 |

Observed categories include:

- Ship Mode: First Class, Same Day, Second Class, Standard Class
- Segment: Consumer, Corporate, Home Office
- Region: Central, East, South, West
- Category: Furniture, Office Supplies, Technology

Whitespace checks were performed on the six categorical columns. No leading or trailing spaces were detected.

**Status:** Completed.

### H. Customer Validation

Customer ID and Customer Name consistency was checked in both directions.

| Metric | Result |
|---|---:|
| Unique Customer IDs | 793 |
| Unique Customer Names | 793 |
| Customer IDs associated with multiple names | 0 |
| Customer names associated with multiple IDs | 0 |

Each customer ID maps to one customer name, and each customer name maps to one customer ID in the source dataset.

**Status:** Completed.

### I. Order Validation

Each `Order_ID` was checked for consistency across:

- Customer ID
- Order Date
- Ship Date
- Ship Mode

No order was found with multiple distinct values for these attributes.

| Metric | Result |
|---|---:|
| Unique Orders | 5,009 |
| Orders with inconsistent customer IDs | 0 |
| Orders with inconsistent order dates | 0 |
| Orders with inconsistent ship dates | 0 |
| Orders with inconsistent ship modes | 0 |

**Status:** Completed.

### J. Product Validation

Product identifiers and names were investigated in both directions.

Initial results:

| Metric | Result |
|---|---:|
| Unique Product IDs | 1,862 |
| Unique Product Names | 1,850 |
| Unique Categories | 3 |
| Unique Sub-Categories | 17 |

**Product ID to Product Name inconsistency:**

32 Product IDs were found to be associated with two different product names each. Their category and subcategory values remained consistent within each ID.

These are distinct product names, not merely formatting or whitespace variations.

**Product Name to Product ID inconsistency:**

The reverse check also identified product names associated with multiple Product IDs.

Examples from the query results include:

| Product Name | Number of Product IDs |
|---|---:|
| Staples | 10 |
| Staple envelope | 9 |
| Easy-staple paper | 8 |
| Staples in misc. colors | 7 |
| Staple remover | 3 |
| Staple holder | 3 |
| Staple magnet | 2 |

Other product names were also associated with multiple IDs.

This means that neither `Product_ID` nor `Product_Name` can currently be treated as a fully reliable unique product key without further investigation.

The raw source values will be preserved. A deliberate product-mapping decision will be made during the cleaning stage before product-level analysis.

**Status:** Completed — inconsistencies identified; cleaning decision pending.

## Initial Audit Conclusions

The initial audit found that:

1. The dataset contains 9,994 rows and 21 columns.
2. No NULL or blank values were detected.
3. `Row_ID` is unique.
4. One potential exact duplicate business record was identified.
5. All tested date and numeric values passed type-conversion checks.
6. No invalid order/ship date sequence was detected.
7. All tested categorical fields have consistent values and no leading/trailing whitespace.
8. Customer ID and Customer Name mappings are consistent.
9. Order attributes are consistent within each order.
10. Product ID and Product Name mappings contain significant inconsistencies.
11. Negative profit values exist and must be preserved for profitability analysis.

## Data Cleaning Plan

The following changes are planned for a separate cleaned table. The raw table `dbo.superstore2` will remain unchanged.

| Area | Planned Action | Reason |
|---|---|---|
| Data types | Convert numeric text fields to INT and DECIMAL | Enable accurate calculations and aggregations |
| Dates | Convert Order Date and Ship Date to DATE using the correct date format | Support time-based analysis |
| Postal Code | Keep as text | Preserve formatting and possible leading zeros |
| Customer fields | Preserve validated Customer ID and Customer Name mappings | No correction currently required |
| Order fields | Preserve validated order attributes | No inconsistency detected |
| Exact duplicate | Investigate the two identical business records before deciding whether to exclude one | Avoid accidental data loss |
| Product mapping | Investigate conflicting Product ID and Product Name relationships | Prevent misleading product-level analysis |
| Negative Profit | Preserve negative values | They represent business losses, not invalid data |
| Discount | Preserve the original decimal rate | Values are already represented correctly |

## Audit Progress

| Audit Section | Status |
|---|---|
| A. Dataset Structure | Completed |
| B. Missing Values | Completed |
| C. Duplicates & Identifiers | Completed |
| D. Data Types & Conversion | Completed |
| E. Date Validation | Completed |
| F. Numeric Validation | Completed |
| G. Categorical Consistency | Completed |
| H. Customer Validation | Completed |
| I. Order Validation | Completed |
| J. Product Validation | Completed |
| K. Geographic Validation | Pending |
| L. Business Logic | Pending |
| M. Outlier Analysis | Pending |
| N. Final Audit Summary | Pending |

**Project Status:** In Progress — Initial dataset inspection and audit sections A–J completed. Geographic validation, business logic checks, outlier analysis, final cleaning decisions, and the cleaned table are still pending.

## Next Steps

1. Complete Geographic Validation.
2. Perform Business Logic checks.
3. Investigate potential outliers.
4. Prepare the final audit summary and cleaning decisions.
5. Create a separate cleaned table with appropriate SQL data types.
6. Begin customer-level analysis:
   - Customer Segmentation
   - New vs. Returning Customers
   - Customer Value
   - Customer Profitability
   - Retention and Inactivity-Based Churn Risk
   - RFM Analysis
7. Build the Power BI dashboard.
8. Document final business insights and recommendations.
