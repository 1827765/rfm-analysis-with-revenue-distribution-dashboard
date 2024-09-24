
# Sales Data Analysis with SQL

This project dives into a dataset related to sales and provides multiple insights, including sales by product lines, yearly trends, territory performance, customer segmentation, and identifying the most frequently sold products together. The project is structured to answer business questions about sales and customer behavior.

## Dataset

The dataset `sales_data_sample` is utilized for this analysis, covering:
- Sales grouped by product line, year, deal size, territory, and country.
- Revenue performance per month, year, and specific product line.
- Customer segmentation using the RFM (Recency, Frequency, Monetary) model.

## SQL Query Breakdown

### 1. **Inspecting the Data**

The dataset is explored by checking distinct values from various columns such as `STATUS`, `YEAR_ID`, `PRODUCTLINE`, `COUNTRY`, `DEALSIZE`, and `TERRITORY`.

```sql
SELECT DISTINCT STATUS FROM sales_data_sample;
SELECT DISTINCT YEAR_ID FROM sales_data_sample;
SELECT DISTINCT PRODUCTLINE FROM sales_data_sample;
SELECT DISTINCT COUNTRY FROM sales_data_sample;
SELECT DISTINCT DEALSIZE FROM sales_data_sample;
SELECT DISTINCT TERRITORY FROM sales_data_sample;
```

### 2. **Total Sales by Product Line**

We group the sales by product lines to identify the highest revenue-generating products.

```sql
SELECT PRODUCTLINE, SUM(SALES) AS TOTAL_SALES
FROM sales_data_sample
GROUP BY PRODUCTLINE
ORDER BY TOTAL_SALES DESC;
```

**Key Findings**:
- **Classic Cars** is the product line with the highest revenue.

### 3. **Total Sales by Year**

The sales are grouped by `YEAR_ID` to see which year generated the most revenue.

```sql
SELECT YEAR_ID, SUM(SALES) AS TOTAL_SALES
FROM sales_data_sample
GROUP BY YEAR_ID
ORDER BY TOTAL_SALES DESC;
```

**Key Findings**:
- **2004** was the year with the highest sales, likely because it had a full year of operation compared to partial operations in other years.

### 4. **Total Sales by Deal Size**

Sales are grouped by `DEALSIZE` to identify which deal sizes bring in the most revenue.

```sql
SELECT DEALSIZE, SUM(SALES) AS TOTAL_SALES
FROM sales_data_sample
GROUP BY DEALSIZE
ORDER BY TOTAL_SALES DESC;
```

### 5. **Total Sales by Territory and Country**

We also analyze sales by `TERRITORY` and `COUNTRY` to understand which regions contribute the most to sales.

```sql
SELECT TERRITORY, SUM(SALES) AS TOTAL_SALES
FROM sales_data_sample
GROUP BY TERRITORY
ORDER BY TOTAL_SALES DESC;

SELECT COUNTRY, SUM(SALES) AS TOTAL_SALES
FROM sales_data_sample
GROUP BY COUNTRY
ORDER BY TOTAL_SALES DESC;
```

### 6. **Best Month for Sales in a Specific Year**

This query shows the best-performing month in terms of sales and order frequency for a selected year.

```sql
SELECT MONTH_ID, SUM(SALES) AS TOTAL_SALES, COUNT(ORDERNUMBER) AS ORDER_FREQUENCY
FROM sales_data_sample
WHERE YEAR_ID = 2004
GROUP BY MONTH_ID
ORDER BY TOTAL_SALES DESC;
```

**Key Findings**:
- In **2004**, November was the best month in terms of both sales and order frequency.

### 7. **Customer Segmentation Using RFM Model**

The RFM (Recency, Frequency, Monetary) model helps segment customers based on their purchase behavior. We first create a temporary table `#rfm` to store this information.

```sql
WITH RFM AS (
    SELECT CUSTOMERNAME,
           SUM(SALES) AS TOTAL_SALE,
           AVG(SALES) AS AVG_SALE,
           COUNT(ORDERNUMBER) AS ORDER_FREQ,
           MAX(ORDERDATE) AS LAST_ORDER_DATE,
           DATEDIFF(DD, MAX(ORDERDATE), (SELECT MAX(ORDERDATE) FROM sales_data_sample)) AS RECENCY
    FROM sales_data_sample
    GROUP BY CUSTOMERNAME
),
RFM_CALC AS (
    SELECT R.*,
           NTILE(4) OVER (ORDER BY RECENCY DESC) AS RFM_RECENCY,
           NTILE(4) OVER (ORDER BY ORDER_FREQ) AS RFM_FREQ,
           NTILE(4) OVER (ORDER BY TOTAL_SALE) AS RFM_MONETARY
    FROM RFM R
)
SELECT C.*, 
       RFM_RECENCY + RFM_MONETARY + RFM_FREQ AS RFM_CELL,
       CAST(RFM_RECENCY AS varchar) + CAST(RFM_MONETARY AS varchar) + CAST(RFM_FREQ AS varchar) AS RFM_CELL_STRING
INTO #rfm
FROM RFM_CALC C;
```

This allows segmentation into categories such as `Lost Customers`, `Active Customers`, and `Loyal Customers`.

### 8. **Frequently Sold Products Together**

We analyze which products are often bought together by customers.

```sql
SELECT DISTINCT ORDERNUMBER, STUFF((
    SELECT ',' + CAST(PRODUCTCODE AS nvarchar)
    FROM sales_data_sample P
    WHERE P.ORDERNUMBER = S.ORDERNUMBER
    FOR XML PATH('')), 1, 1, '')
FROM sales_data_sample S
ORDER BY 2 DESC;
```

## Conclusion

This project offers valuable insights into sales performance by product line, customer behavior, and trends by year. Additionally, the RFM model helps businesses understand customer loyalty and create targeted marketing strategies.
