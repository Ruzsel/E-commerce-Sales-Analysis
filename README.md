# E-commerce dataset
## Overview
The data used is derived from a purely fictional e-commerce platform.
Regarding the dataset explanation, it goes as follows:

### order_detail:

| Column Name       | Description                                                                                      |
|-------------------|--------------------------------------------------------------------------------------------------|
| id                | Unique number of the order / order ID.                                                           |
| customer_id       | Unique number of the customer.                                                                   |
| order_date        | Date when the transaction was made.                                                              |
| sku_id            | Unique number of the product (SKU stands for Stock Keeping Unit).                                 |
| price             | Price listed on the price tag.                                                                   |
| qty_ordered       | Quantity of items purchased by the customer.                                                      |
| before_discount   | Total price value of the product (price * qty_ordered).                                           |
| discount_amount   | Total discount amount of the product.                                                             |
| after_discount    | Total price value of the product after discount.                                                  |
| is_gross          | Indicates whether the customer has not paid the order.                                            |
| is_valid          | Indicates whether the customer has made the payment.                                              |
| is_net            | Indicates whether the transaction is completed.                                                   |
| payment_id        | Unique number of the payment method.                

### sku_detail:

| Column Name | Description                                                   |
|-------------|---------------------------------------------------------------|
| id          | Unique number of the product (can be used as a key when joining).  |
| sku_name    | Name of the product.                                         |
| base_price  | Price of the item listed on the price tag.                    |
| cogs        | Cost of goods sold / total cost to sell 1 product.            |
| category    | Product category.                                            |

### customer_detail:

| Column Name     | Description                                                          |
|-----------------|----------------------------------------------------------------------|
| id              | Unique number of the customer.                                      |
| registered_date | Date when the customer started registering as a member.              |

### Payment_detail:

| Column Name     | Description                                      |
|-----------------|--------------------------------------------------|
| id              | Unique number of the payment method.            |
| payment_method  | Payment method used.                             |

## Project Explanation
The purpose of this project is to answer 5 business questions based on the available dataset.

## Tools Used
- PostgreSQL
- PGAdmin 4

# Questions
## 1. During transactions that occurred in 2021, in which month did the total transaction value (after discount) reach its highest?
```sql
SELECT 
    EXTRACT(MONTH FROM order_date) AS month,
    ROUND(SUM(after_discount::numeric), 2) AS total_transaction
FROM 
    order_detail
WHERE 
    is_valid = 1
    AND EXTRACT(YEAR FROM order_date) = 2021
GROUP BY  
    EXTRACT(MONTH FROM order_date)
ORDER BY 
    total_transaction DESC;
```
<img width="212" alt="1" src="https://github.com/Ruzsel/ecommerce-dataset/assets/150054552/7aeee9b9-02e3-4a94-8dc8-e5e6af7bb898">

## 2. During transactions in the year 2022, which category generated the highest transaction value?
```sql
SELECT
    category,
    ROUND(SUM(after_discount)::numeric, 2) AS total_transaction
FROM
    order_detail
INNER JOIN
    sku_detail
ON
    order_detail.sku_id = sku_detail.id
WHERE 
    order_detail.is_valid = 1
    AND EXTRACT(YEAR FROM order_date) = 2022
GROUP BY
    category
ORDER BY
    total_transaction DESC
LIMIT 10;
```
<img width="272" alt="2" src="https://github.com/Ruzsel/ecommerce-dataset/assets/150054552/dcb506f0-4183-4ef7-92e4-df1d89b1542b">

## 3. Bandingkan nilai transaksi dari masing-masing kategori pada tahun 2021 dengan 2022.
Sebutkan kategori apa saja yang mengalami peningkatan dan kategori apa yang mengalami
penurunan nilai transaksi dari tahun 2021 ke 2022.
```sql
WITH category_total_tran AS (
    SELECT 
        sku_detail.category,
        EXTRACT(YEAR FROM order_detail.order_date) AS "year",
        SUM(order_detail.after_discount) AS total_transaction
    FROM 
        order_detail
    INNER JOIN 
        sku_detail
    ON order_detail.sku_id = sku_detail.id
    WHERE 
        order_detail.is_valid = 1
        AND EXTRACT(YEAR FROM order_detail.order_date) IN (2021, 2022)
    GROUP BY  
        sku_detail.category, 
        "year"
)
SELECT
    category,
    MAX(CASE WHEN "year" = 2021 THEN total_transaction END) AS total_transaction_2021,
    MAX(CASE WHEN "year" = 2022 THEN total_transaction END) AS total_transaction_2022,
    CASE
        WHEN MAX(CASE WHEN "year" = 2022 THEN total_transaction END) > MAX(CASE WHEN "year" = 2021 THEN total_transaction END) THEN 'Peningkatan'
        WHEN MAX(CASE WHEN "year" = 2022 THEN total_transaction END) < MAX(CASE WHEN "year" = 2021 THEN total_transaction END) THEN 'Penurunan'
        ELSE 'Tidak Berubah'
    END AS trend
FROM
    category_total_tran
GROUP BY
    category
;
```
<img width="403" alt="3" src="https://github.com/Ruzsel/ecommerce-dataset/assets/150054552/2253871e-6de2-4be1-bb3d-f1dbb0446305">

## 4. Display the top 5 most popular payment methods used during 2022 (based on total unique orders).
```sql
SELECT
    payment_method,
    COUNT(DISTINCT order_detail.id) AS total_unique_orders
FROM
    order_detail
INNER JOIN
    payment_detail
ON
	order_detail.payment_id = payment_detail.id
WHERE
    order_detail.is_valid = 1
    AND EXTRACT(YEAR FROM order_detail.order_date) = 2022
GROUP BY
    payment_method
ORDER BY
    total_unique_orders DESC
LIMIT 5;
```
<img width="346" alt="4" src="https://github.com/Ruzsel/ecommerce-dataset/assets/150054552/39d85295-1349-4249-a6e7-af5287af8e36">

## 5. Sort these products based on their transaction values, starting from the 5th product:
1. Lenovo
2. Huawei
3. Sony
4. Apple
5. Samsung
```sql
SELECT
    product_names,
    SUM(after_discount) AS total_transaction_value
FROM (
    SELECT
        CASE 
            WHEN UPPER(sku_detail.sku_name) LIKE '%SAMSUNG%' THEN 'Samsung'
            WHEN UPPER(sku_detail.sku_name) LIKE '%IPHONE%'
                OR UPPER(sku_detail.sku_name) LIKE '%MACBOOK%'
                OR UPPER(sku_detail.sku_name) LIKE '%APPLE%' THEN 'Apple'
			WHEN UPPER(sku_detail.sku_name) LIKE '%SONY%' THEN 'Sony'
			WHEN UPPER(sku_detail.sku_name) LIKE '%HUAWEI%' THEN 'Huawei'
			WHEN UPPER(sku_detail.sku_name) LIKE '%LENOVO%' THEN 'Lenovo'
        END AS product_names,
        order_detail.after_discount
    FROM
        sku_detail
    JOIN
        order_detail ON sku_detail.id = order_detail.sku_id
    WHERE
        order_detail.is_valid = 1
        AND sku_detail.sku_name IS NOT NULL
) AS subquery
GROUP BY
    product_names
HAVING
    product_names IS NOT NULL
ORDER BY
    total_transaction_value DESC
;
```
<img width="355" alt="5" src="https://github.com/Ruzsel/ecommerce-dataset/assets/150054552/aafc1162-45cc-444f-ba2f-e087f3e54a2d">
