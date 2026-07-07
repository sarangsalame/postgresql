# PostgreSQL Notes

A quick, practical reference for common **PostgreSQL** SQL commands — creating and modifying tables, inserting and querying data, and building a small sample database.

> PostgreSQL is a free, open-source, object-relational database system. SQL keywords are case-insensitive (`SELECT` = `select`), but the conventions below use uppercase for keywords to improve readability.

---

## Table of Contents

- [Working with a Single Table](#working-with-a-single-table)
  - [Create a Table](#create-a-table)
  - [Select Data](#select-data)
  - [Insert a Single Row](#insert-a-single-row)
  - [Insert Multiple Rows](#insert-multiple-rows)
  - [Add a Column (ALTER TABLE ... ADD)](#add-a-column-alter-table--add)
  - [Update Rows (UPDATE)](#update-rows-update)
  - [Change a Column's Data Type (ALTER COLUMN)](#change-a-columns-data-type-alter-column)
  - [Change Maximum Allowed Characters](#change-maximum-allowed-characters)
  - [Drop a Column](#drop-a-column)
  - [Delete Rows (DELETE)](#delete-rows-delete)
  - [Empty a Table (TRUNCATE)](#empty-a-table-truncate)
  - [Drop a Table](#drop-a-table)
- [Building a Sample Database](#building-a-sample-database)
  - [categories](#1-categories)
  - [customers](#2-customers)
  - [products](#3-products)
  - [orders](#4-orders)
  - [order_details](#5-order_details)
  - [testproducts](#6-testproducts)
- [Useful Query Examples](#useful-query-examples)

---

## Working with a Single Table

The examples in this section use a simple `cars` table.

### Create a Table

Use `CREATE TABLE` to define a new table and its columns. Each column needs a name and a data type.

```sql
CREATE TABLE cars (
  brand VARCHAR(255),
  model VARCHAR(255),
  year INT
);
```

| Data Type      | Description                                  |
| -------------- | -------------------------------------------- |
| `VARCHAR(n)`   | Variable-length text, up to `n` characters   |
| `INT`          | Whole number (integer)                       |
| `DECIMAL(p,s)` | Fixed-precision number (`p` digits, `s` decimals) |
| `DATE`         | Calendar date (`YYYY-MM-DD`)                 |
| `SERIAL`       | Auto-incrementing integer (great for IDs)    |

### Select Data

Use `SELECT` to read rows from a table. `*` returns **all** columns.

```sql
SELECT * FROM cars;
```

### Insert a Single Row

`INSERT INTO` adds a new row. List the columns, then supply matching `VALUES`.

```sql
INSERT INTO cars (brand, model, year)
VALUES ('Ford', 'Mustang', 1964);
```

**Result:** `INSERT 0 1` — the `1` means one row was inserted.

### Insert Multiple Rows

Separate each row of values with a comma.

```sql
INSERT INTO cars (brand, model, year)
VALUES
  ('Volvo', 'p1800', 1968),
  ('BMW', 'M1', 1978),
  ('Toyota', 'Celica', 1975);
```

**Result:** `INSERT 0 3` — three rows were successfully inserted.

### Add a Column (ALTER TABLE ... ADD)

`ALTER TABLE` adds, deletes, or modifies columns in an existing table. Here we add a `color` column.

```sql
ALTER TABLE cars
ADD color VARCHAR(255);
```

### Update Rows (UPDATE)

`UPDATE` modifies values in existing rows. The `WHERE` clause decides **which** rows change.

```sql
UPDATE cars
SET color = 'red'
WHERE brand = 'Volvo';
```

To update more than one column, separate the `column = value` pairs with commas:

```sql
UPDATE cars
SET color = 'white', year = 1970
WHERE brand = 'Toyota';
```

> ⚠️ **Warning:** Without a `WHERE` clause, **every** row in the table is updated.

### Change a Column's Data Type (ALTER COLUMN)

Change the data type of an existing column. Below, `year` is converted from `INT` to `VARCHAR(4)`.

```sql
ALTER TABLE cars
ALTER COLUMN year TYPE VARCHAR(4);
```

### Change Maximum Allowed Characters

Adjust the maximum length of a text column. Here the `color` column is capped at 30 characters.

```sql
ALTER TABLE cars
ALTER COLUMN color TYPE VARCHAR(30);
```

### Drop a Column

Remove a column entirely with `ALTER TABLE ... DROP COLUMN`.

```sql
ALTER TABLE cars
DROP COLUMN color;
```

### Delete Rows (DELETE)

`DELETE` removes existing rows. Use `WHERE` to target specific rows.

```sql
DELETE FROM cars
WHERE brand = 'Volvo';
```

Delete **all** rows (the table structure remains):

```sql
DELETE FROM cars;
```

### Empty a Table (TRUNCATE)

`TRUNCATE TABLE` removes all rows at once. It is faster than `DELETE` for clearing a whole table and achieves the same result as `DELETE` without a `WHERE` clause.

```sql
TRUNCATE TABLE cars;
```

> **DELETE vs TRUNCATE:** `DELETE` removes rows one by one and can be filtered with `WHERE`; `TRUNCATE` wipes the entire table in one operation and cannot be filtered.

### Drop a Table

`DROP TABLE` removes the table and all of its data permanently.

```sql
DROP TABLE cars;
```

> ⚠️ This is irreversible — the table definition and every row are gone.

---

## Building a Sample Database

The following six related tables model a small store (a Northwind-style sample):

| Table           | Purpose                                          |
| --------------- | ------------------------------------------------ |
| `categories`    | Product categories (e.g. Beverages, Seafood)     |
| `customers`     | Customer contact and address details             |
| `products`      | Products for sale, linked to a category          |
| `orders`        | Orders placed by customers                        |
| `order_details` | Line items linking orders to products             |
| `testproducts`  | Scratch table used for join/testing examples     |

**Relationships:** `products.category_id → categories.category_id`, `orders.customer_id → customers.customer_id`, and `order_details` links `orders.order_id` with `products.product_id`.

### 1. categories

```sql
CREATE TABLE categories (
  category_id SERIAL NOT NULL PRIMARY KEY,
  category_name VARCHAR(255),
  description VARCHAR(255)
);

INSERT INTO categories (category_name, description)
VALUES
  ('Beverages', 'Soft drinks, coffees, teas, beers, and ales'),
  ('Condiments', 'Sweet and savory sauces, relishes, spreads, and seasonings'),
  ('Confections', 'Desserts, candies, and sweet breads'),
  ('Dairy Products', 'Cheeses'),
  ('Grains/Cereals', 'Breads, crackers, pasta, and cereal'),
  ('Meat/Poultry', 'Prepared meats'),
  ('Produce', 'Dried fruit and bean curd'),
  ('Seafood', 'Seaweed and fish');
```

### 2. customers

```sql
CREATE TABLE customers (
  customer_id SERIAL NOT NULL PRIMARY KEY,
  customer_name VARCHAR(255),
  contact_name VARCHAR(255),
  address VARCHAR(255),
  city VARCHAR(255),
  postal_code VARCHAR(255),
  country VARCHAR(255)
);
```

Insert the customer rows (91 in total). A sample is shown below — the full data set follows the same pattern:

```sql
INSERT INTO customers (customer_name, contact_name, address, city, postal_code, country)
VALUES
  ('Alfreds Futterkiste', 'Maria Anders', 'Obere Str. 57', 'Berlin', '12209', 'Germany'),
  ('Ana Trujillo Emparedados y helados', 'Ana Trujillo', 'Avda. de la Constitucion 2222', 'Mexico D.F.', '05021', 'Mexico'),
  ('Antonio Moreno Taquera', 'Antonio Moreno', 'Mataderos 2312', 'Mexico D.F.', '05023', 'Mexico'),
  ('Around the Horn', 'Thomas Hardy', '120 Hanover Sq.', 'London', 'WA1 1DP', 'UK'),
  ('Berglunds snabbkoep', 'Christina Berglund', 'Berguvsvegen 8', 'Lulea', 'S-958 22', 'Sweden');
  -- ... continue through all 91 customers ...
```

> The `customer_id` column is `SERIAL`, so it is generated automatically — you do not supply it in the `INSERT`.

### 3. products

```sql
CREATE TABLE products (
  product_id SERIAL NOT NULL PRIMARY KEY,
  product_name VARCHAR(255),
  category_id INT,
  unit VARCHAR(255),
  price DECIMAL(10, 2)
);
```

Insert the 77 products. Because explicit `product_id` values are supplied here, the values are listed for each row:

```sql
INSERT INTO products (product_id, product_name, category_id, unit, price)
VALUES
  (1, 'Chais', 1, '10 boxes x 20 bags', 18),
  (2, 'Chang', 1, '24 - 12 oz bottles', 19),
  (3, 'Aniseed Syrup', 2, '12 - 550 ml bottles', 10),
  (4, 'Chef Antons Cajun Seasoning', 2, '48 - 6 oz jars', 22),
  (5, 'Chef Antons Gumbo Mix', 2, '36 boxes', 21.35);
  -- ... continue through product_id 77 ...
```

> `category_id` links each product to a row in `categories`. `price` uses `DECIMAL(10, 2)` so values keep two decimal places.

### 4. orders

```sql
CREATE TABLE orders (
  order_id SERIAL NOT NULL PRIMARY KEY,
  customer_id INT,
  order_date DATE
);
```

Insert the orders (IDs `10248`–`11077`). Each order references a customer and a date:

```sql
INSERT INTO orders (order_id, customer_id, order_date)
VALUES
  (10248, 90, '2021-07-04'),
  (10249, 81, '2021-07-05'),
  (10250, 34, '2021-07-08'),
  (10251, 84, '2021-07-08'),
  (10252, 76, '2021-07-09');
  -- ... continue through order_id 11077 ...
```

> `customer_id` links each order to a row in `customers`. Dates use the ISO format `YYYY-MM-DD`.

### 5. order_details

```sql
CREATE TABLE order_details (
  order_detail_id SERIAL NOT NULL PRIMARY KEY,
  order_id INT,
  product_id INT,
  quantity INT
);
```

This is a **junction (link) table**: each row represents one product line within an order, so a single order can have several rows.

```sql
INSERT INTO order_details (order_id, product_id, quantity)
VALUES
  (10248, 11, 12),
  (10248, 42, 10),
  (10248, 72, 5),
  (10249, 14, 9),
  (10249, 51, 40);
  -- ... continue for all order line items ...
```

> Here order `10248` contains three line items (products `11`, `42`, and `72`). `order_id` links to `orders` and `product_id` links to `products`.

### 6. testproducts

A small scratch table used to demonstrate joins (including rows that do **not** match a category).

```sql
CREATE TABLE testproducts (
  testproduct_id SERIAL NOT NULL PRIMARY KEY,
  product_name VARCHAR(255),
  category_id INT
);

INSERT INTO testproducts (product_name, category_id)
VALUES
  ('Johns Fruit Cake', 3),
  ('Marys Healthy Mix', 9),
  ('Toms Salty Sausage', 1),
  ('Jims Big Blowout', 12);
```

> Category IDs `9` and `12` do not exist in `categories` — this is intentional, so you can see how `INNER JOIN` (drops non-matches) differs from `LEFT JOIN` (keeps them).

---

## Useful Query Examples

Once the sample database is populated, these queries show how the tables work together.

**List products with their category name (INNER JOIN):**

```sql
SELECT products.product_name, categories.category_name
FROM products
INNER JOIN categories ON products.category_id = categories.category_id;
```

**Count orders per customer:**

```sql
SELECT customers.customer_name, COUNT(orders.order_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_name
ORDER BY order_count DESC;
```

**Total quantity sold per product:**

```sql
SELECT p.product_name, SUM(od.quantity) AS total_sold
FROM order_details od
INNER JOIN products p ON od.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_sold DESC;
```

**Filter with WHERE and sort with ORDER BY:**

```sql
SELECT product_name, price
FROM products
WHERE price > 50
ORDER BY price DESC;
```

---

*Reference notes for learning PostgreSQL. Run these statements with the `psql` command-line client or any PostgreSQL GUI (e.g. pgAdmin, DBeaver).*

