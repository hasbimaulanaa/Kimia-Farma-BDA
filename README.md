# Kimia-Farma-BDA

4. Fixing Formatting
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
