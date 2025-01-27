![](https://github.com/hasbimaulanaa/Portoflio/blob/main/farma.png)
# Virtual Internship Experience: Big Data Analytics - Kimia Farma
VIX Big Data Analytics Kimia Farma merupakan virtual internship experience yang difasilitasi oleh Rakamin Academy. Pada project ini saya berperan sebagai Data Analyst Intern yang diminta untuk menganalisis dan membuat laporan penjualan perusahaan menggunakan data-data yang telah disediakan. Dari project ini, saya juga banyak belajar tentang data warehouse, dataleke, dan datamart.

VIX Big Data Analytics Kimia Farma merupakan virtual internship experience yang difasilitasi oleh [Rakamin Academy](https://www.rakamin.com/virtual-internship-experience/kimiafarma-big-data-analytics-virtual-internship-program). Pada project ini saya berperan sebagai Data Analyst Intern yang diminta untuk menganalisis dan membuat laporan penjualan perusahaan menggunakan data-data yang telah disediakan. Dari project ini, saya juga banyak belajar tentang data data warehouse, dataleke, dan datamart. <br>
<br>
# Project Description
The goal of this project is to provide strategic insights to improve operational efficiency through data analysis.
Objectives:
1. How did Kimia Farma's branches perform in terms of sales, profit, and ratings from 2020–2023?
2. What trends emerged in Kimia Farma's sales, discounts, and profits during this period?
3. Which branch contributed the most to total transactions and net sales?
4. How is profit distributed across provinces in Indonesia?
5. What recommendations can be made based on this analysis to improve business performance?
# Datasets Used
- kf_final_transaction.csv: Contains transaction data, including transaction ID, date, customer name, product price, discounts, and customer ratings.
- kf_product.csv: Provides product details, such as product ID, name, category, and price.
- kf_inventory.csv: Contains inventory data for branch offices.
- kf_kantor_cabang.csv: Includes branch office data such as location, category, and customer ratings.
  
# Process
- Perform data cleaning
```{r}
DECLARE query STRING;

-- Buat query dinamis untuk semua tabel
SET query = (
  SELECT STRING_AGG(
    FORMAT("""
      SELECT '%s' AS table_name, 
             '%s' AS column_name, 
             COUNTIF(%s IS NULL) AS null_count
      FROM `kimia_farma_big_data.%s`
    """, table_name, column_name, column_name, table_name),
    " UNION ALL "
  )
  FROM `kimia_farma_big_data.INFORMATION_SCHEMA.COLUMNS`
  WHERE table_name IN ('kf_final_transaction','kf_inventory', 'kf_kantor_cabang', 'kf_product')
);

-- Eksekusi query untuk menghitung NULL di semua tabel dan kolom
EXECUTE IMMEDIATE query;
```
![](https://github.com/hasbimaulanaa/Portoflio/blob/main/null.png)

# Analyze
- create an analysis table from datasets
```{r}
-- Membuat tabel analisa
CREATE TABLE `kimi-farma-project.kimia_farma_big_data.tabel_analisa` AS
WITH transaction_data AS (
  SELECT 
    t.transaction_id,
    t.date,
    t.branch_id,
    c.branch_name,
    c.kota,
    c.provinsi,
    c.rating AS rating_cabang,
    t.customer_name,
    t.product_id,
    p.product_name,
    t.price AS actual_price,
    t.discount_percentage,
    -- Menghitung persentase gross laba
    CASE
      WHEN t.price <= 50000 THEN 0.10
      WHEN t.price > 50000 AND t.price <= 100000 THEN 0.15
      WHEN t.price > 100000 AND t.price <= 300000 THEN 0.20
      WHEN t.price > 300000 AND t.price <= 500000 THEN 0.25
      WHEN t.price > 500000 THEN 0.30
    END AS persentase_gross_laba,
    -- Menghitung nett sales (harga setelah diskon)
    t.price * (1 - t.discount_percentage / 100) AS nett_sales,
    -- Menghitung nett profit (harga setelah diskon * persentase laba)
    t.price * (1 - t.discount_percentage / 100) *
    CASE
      WHEN t.price <= 50000 THEN 0.10
      WHEN t.price > 50000 AND t.price <= 100000 THEN 0.15
      WHEN t.price > 100000 AND t.price <= 300000 THEN 0.20
      WHEN t.price > 300000 AND t.price <= 500000 THEN 0.25
      WHEN t.price > 500000 THEN 0.30
    END AS nett_profit,
    t.rating AS rating_transaksi
  FROM `kimi-farma-project.kimia_farma_big_data.kf_final_transaction` t
  LEFT JOIN `kimi-farma-project.kimia_farma_big_data.kf_product` p
    ON t.product_id = p.product_id
  LEFT JOIN `kimi-farma-project.kimia_farma_big_data.kf_kantor_cabang` c
    ON t.branch_id = c.branch_id
)
SELECT *
FROM transaction_data;

```
![](https://github.com/hasbimaulanaa/Portoflio/blob/main/hasil3.png)

# Data Visualization
- Kimia Farma Performance analytics Dashboard
  [Click to view dashboard](https://lookerstudio.google.com/reporting/0d4ff341-69ea-4b99-8c21-11518f9bd32d)
  
![](https://github.com/hasbimaulanaa/Portoflio/blob/main/hasil%204.png)

# Dashboard Insights
1. Financial and Transaction Performance (2020-2023):
- Net Sales: Consistent sales performance with annual sales between IDR 86.5B and IDR 87.1B.
- Net Profit: Stable profit trends, with an average annual profit of IDR 24.6B.
- Transactions: Over 672K transactions recorded, demonstrating consistent customer activity.
2. Regional Performance:
- Top Provinces by Net Sales:
  * West Java: IDR 102.5B, the highest revenue contributor.
  * North Sumatra and Central Java: Other high-performing regions.
  * East Kalimantan and Riau: Underperforming regions with sales below IDR 11B.
- Total Profit by Province:
  * The heatmap indicates that West Java leads in profitability, while other regions exhibit varied performance levels.
3. Branch Performance:
- Top 5 Branches by Net Sales:
  * Kimia Farma Klinik-Apotek Lab Subang: IDR 5B.
  * Other branches in Sumedang, Garut, and Sukabumi also contributed significantly, with sales over IDR 4.5B each.
4. Product Insights:
- Top 3 Product Transactions:
  * Psychoactive Drugs & Hypnotics dominate sales volume.
  * Other significant categories include Anxiolytic Drugs and Drugs for Obstructive Airway Diseases.
-  Highest Margin Products:
   * Products in Psychoactive Drugs & Hypnotics account for 16.6% of the total margin, making them a strategic focus for profitability.

# Recomendations
1. Focus on expanding high-margin products like psychoactive drugs.
2. Strengthen market presence in West Java, especially leveraging high-performing branches.
3. Tailor marketing strategies for North Sumatra and Central Java to capitalize on existing opportunities.
4. Explore promotional programs in underperforming regions like East Kalimantan and Riau to drive growth.
   
