# Week 40 — Exercises: SQL Fundamentals

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task

This week you'll build the TrailShop database from scratch and practice manipulating data.

### Task 1.1: Create the Database

1. Open your PostgreSQL terminal (psql) or pgAdmin
2. Create a new database called `trailshop`
3. Connect to it

### Task 1.2: Create All Tables

Write and execute the CREATE TABLE statements for all six TrailShop tables in the correct order:

- categories
- customers
- products
- product_categories
- orders
- order_items

**Requirements:**

- Use appropriate data types for each column
- Include all constraints from the theory (NOT NULL, UNIQUE, CHECK, FOREIGN KEY, DEFAULT)
- Use SERIAL for primary keys
- Ensure foreign keys reference the correct parent tables

**Verify** by running `\dt` in psql to list all tables.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
> DROP TABLE IF EXISTS order_items CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS product_categories CASCADE;
DROP TABLE IF EXISTS products CASCADE;
DROP TABLE IF EXISTS customers CASCADE;
DROP TABLE IF EXISTS categories CASCADE;

CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE product_categories (
    product_id INTEGER NOT NULL REFERENCES products(product_id) ON DELETE CASCADE,
    category_id INTEGER NOT NULL REFERENCES categories(category_id) ON DELETE CASCADE,
    PRIMARY KEY (product_id, category_id)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id) ON DELETE CASCADE,
    status VARCHAR(50) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled')),
    order_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(product_id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0)
);
> 
>
>
> ```

### Task 1.3: Insert Sample Data

Insert the following data:

**Categories** (at least 5):

- Footwear, Backpacks, Tents, Clothing, Accessories

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>INSERT INTO categories (name, description) VALUES
('Footwear', 'Hiking boots, trail shoes and socks'),
('Backpacks', 'Daypacks and multi-day packs'),
('Tents', 'Shelters and tents'),
('Clothing', 'Jackets, pants and base layers'),
('Accessories', 'Water bottles, tools and gear');
>
> ```

**Customers** (at least 5):

- Use easy to write names with realistic email addresses

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO customers (first_name, last_name, email) VALUES
('Matti', 'Virtanen', 'matti.virtanen@example.fi'),
('Anna', 'Korhonen', 'anna.korhonen@example.fi'),
('Juho', 'Mäkelä', 'juho.makela@example.fi'),
('Laura', 'Niemi', 'laura.niemi@example.fi'),
('Ville', 'Koskinen', 'ville.koskinen@example.fi');
>
>
> ```

**Products** (at least 10):

- At least 2 products per category
- At least one product assigned to **two or more** categories
- Prices ranging from €20 to €500
- Various stock levels

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO products (name, description, price, stock) VALUES
('Trail Trekker Boots', 'Durable waterproof hiking boots', 189.99, 25),
('Alpine Runner Shoes', 'Lightweight trail running shoes', 129.50, 40),
('Summit 45L Pack', 'Technical backpack for multi-day trips', 159.00, 15),
('Daylite 15L Pack', 'Compact daypack for short hikes', 49.99, 50),
('Ultralight 2P Tent', '2-person 3-season camping tent', 349.99, 10),
('Basecamp 4P Tent', 'Spacious family camping tent', 499.00, 5),
('Thermal Base Top', 'Merino wool long-sleeve top', 79.99, 30),
('Rainproof Shell Jacket', 'Waterproof windbreaker jacket', 199.99, 20),
('Trek Pole Pair', 'Aluminum collapsible trekking poles', 45.00, 60),
('HydroFlask 1L', 'Insulated stainless steel water bottle', 35.00, 100);
>
>
> ```

**Product categories:**

- Insert rows into `product_categories` so every sample product is linked to at least one category

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO product_categories (product_id, category_id) VALUES
(1, 1), (2, 1), (3, 2), (4, 2), (5, 3), 
(6, 3), (7, 4), (8, 4), (8, 5), (9, 5), (10, 5);
>
>
> ```

**Orders** (at least 5):

- Different customers, different statuses

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO orders (customer_id, status) VALUES
(1, 'delivered'),
(2, 'shipped'),
(3, 'pending'),
(4, 'delivered'),
(5, 'cancelled');
>
>
> ```

**Order Items** (at least 10):

- Multiple items in some orders, single items in others

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 189.99), (1, 7, 2, 79.99),
(2, 2, 1, 129.50), (2, 10, 1, 35.00),
(3, 3, 1, 159.00), (3, 9, 1, 45.00), (3, 10, 2, 35.00),
(4, 5, 1, 349.99), (4, 8, 1, 199.99),
(5, 4, 1, 49.99);
>
>
> ```

**Verify** each insert with `SELECT * FROM table_name;`

### Task 1.4: Practice UPDATE

> [!TIP]
> **Recommended practice.** Do Tasks 1.4–1.6. They are not required to finish the TrailShop project. They prepare you for the exams. Task 1.6 renames `stock` to `quantity_in_stock`. Later weeks still use `stock`, so after you practice the rename, change the column name back.

Perform the following updates and verify each one:

1. Increase the price of all products in the Footwear category by 10% (join through `product_categories`)
2. Change customer #3's email to a new address
3. Update the status of order #2 from 'shipped' to 'delivered'
4. Set the stock of 'HydroFlask 1L' to 85
5. Add a description to any product that currently has NULL in description

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> --  1. Increase price of Footwear products by 10%
UPDATE products
SET price = price * 1.10
WHERE product_id IN (
    SELECT pc.product_id 
    FROM product_categories pc
    JOIN categories c ON pc.category_id = c.category_id
    WHERE c.name = 'Footwear'
);

-- 2. Change customer #3's email
UPDATE customers
SET email = 'juho.makela.updated@example.fi'
WHERE customer_id = 3;

-- 3. Update status of order #2
UPDATE orders
SET status = 'delivered'
WHERE order_id = 2;

-- 4. Set stock of 'HydroFlask 1L' to 85
UPDATE products
SET stock = 85
WHERE name = 'HydroFlask 1L';

-- 5. Add description to products where description IS NULL
UPDATE products
SET description = 'Standard outdoor equipment'
WHERE description IS NULL;
>
>
> ```

### Task 1.5: Practice DELETE

1. Delete the most recently created order (and observe what happens to its order_items if you used CASCADE)
2. Try to delete a product that appears in `order_items` — what error do you get?
3. Delete a category that has products linked through `product_categories`. The products should remain; only the link rows should disappear. Confirm this.
4. Delete a customer who has no orders

### Task 1.6: Practice ALTER TABLE

1. Add a column `phone VARCHAR(20)` to the customers table
2. Add a column `weight_grams INTEGER` to the products table
3. Add a CHECK constraint to ensure `weight_grams > 0` (allow NULL though — not all products have weight recorded yet)
4. Rename the `stock` column in products to `quantity_in_stock`

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- 1. Add phone column
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);

-- 2. Add weight_grams column
ALTER TABLE products ADD COLUMN weight_grams INTEGER;

-- 3. Add CHECK constraint on weight_grams
ALTER TABLE products ADD CONSTRAINT chk_weight_positive CHECK (weight_grams > 0);

-- 4. Rename stock to quantity_in_stock
ALTER TABLE products RENAME COLUMN stock TO quantity_in_stock;

-- Rename column back to 'stock' as instructed in the note
ALTER TABLE products RENAME COLUMN quantity_in_stock TO stock;
>
>
> ```

---

## Exercise 2: Theory Review Questions

Answer the following questions in your own words using the answer fields below:

1. What does SQL stand for, and why was the language designed to look like English?

> [!NOTE]
> **_Your Answer_**
>
> _(Structured Query language .It was made English-like so database users don't need deep programming knowledge ,statements read like plain sentences easier to learn, write and read.)_

2. Explain the difference between DDL and DML. Give two example commands for each.

> [!NOTE]
> **_Your Answer_**
>
> _(DDL = Date Definitaion Language - defines/modifies database structure.
> Examples: CREATE TABLE, ALTER TABLE 
>   DML = Data Manipulation Language - reads/modifies the data inside tables.
>  Examples:SELECT,INSERT)
> 

3. What is the difference between DCL and TCL? When would you use each?

> [!NOTE]
> **_Your Answer_**
>
> _(DCL = Data Control language - manages premissions /access rights.
> Use:when granting /revoking who can read / write data - GRANT ,REVOKE.
> TCL = Transction Control Language -manages groups of changes as one unit.
> Use : when multiple edits must all succeed or all fail - COMMIT ,ROLLBACK.  .)_

4. Why must you create tables in a specific order? What determines that order?

> [!NOTE]
> **_Your Answer_**
>
> _(Foreign keys link child tables to parent tables. You must create the referenced
>  (parent) table first, otherwise the foreign key has nowhere to point. Order =
> independent tables first - tables that reference them last..)_

5. What is the difference between a column-level constraint and a table-level constraint? When _must_ you use a table-level constraint?

> [!NOTE]
> **_Your Answer_**
>
> _(Column‑level: defined right after one column; applies only to that column.
> Table‑level: defined separately after all columns; can span multiple columns.
>Must use table‑level when: the constraint involves more than one column.)_

6. Explain the difference between `DELETE FROM products;` and `TRUNCATE TABLE products;`. When would you prefer each?

> [!NOTE]
> **_Your Answer_**
>
> _(DELETE FROM products; - removes rows one by one; can have WHERE; keeps auto‑increment values; can be rolled back.
> TRUNCATE TABLE products; → removes all rows instantly, resets ID counters, faster, but
> no WHERE and cannot easily roll back in all contexts.
> Prefer DELETE when filtering rows; prefer TRUNCATE when clearing everything.)_

7. What does `ON DELETE CASCADE` do on a foreign key? Give a real-world scenario where it's appropriate and one where it would be dangerous.

> [!NOTE]
> **_Your Answer_**
>
> _(When a parent row is deleted, automatically deletes all matching child rows.
> Appropriate: delete an order → delete its line items too.
> Dangerous: delete a product → accidentally delete all orders that ever included it (loses sales history.)_

8. Why should you store `unit_price` in the `order_items` table instead of just looking it up from the `products` table?

> [!NOTE]
> **_Your Answer_**
>
> _(Product prices change over time. If you look up the current price later, it won’t
> match what the customer actually paid. Storing the price at purchase time preserves the
> accurate historical record..)_

9. What is the difference between SERIAL and GENERATED ALWAYS AS IDENTITY? Which would you use in a new project and why?

> [!NOTE]
> **_Your Answer_**
>
> _(SERIAL is older PostgreSQL shorthand that creates a sequence behind the scenes.
> GENERATED ALWAYS AS IDENTITY is the modern SQL‑standard way, clearer and safer.
> For new projects → use IDENTITY; it’s standard, explicit, and preferred.)_

10. Explain why `UPDATE products SET price = 9.99;` is dangerous. What steps should you take before running any UPDATE statement?

> [!NOTE]
> **_Your Answer_**
>
> _(No WHERE clause → updates every single row in the table.
>Before running: write SELECT * FROM … with the same conditions first to preview exactly
> which rows will change; always include a precise WHERE clause.)_

---

## Exercise 3: SQL Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL statements for each task in the **Your SQL** fields below. Verify by running them when ready.

### 3.1 CREATE TABLE

Write a CREATE TABLE statement for a `suppliers` table with the following columns:

- supplier_id (auto-incrementing primary key)
- company_name (required, max 200 characters, must be unique)
- contact_name (max 150 characters)
- email (max 255 characters, required, unique)
- phone (max 20 characters)
- country (max 100 characters, required, default 'Finland')

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- CREATE TABLE suppliers (
    supplier_id SERIAL PRIMARY KEY,
    company_name VARCHAR(200) NOT NULL UNIQUE,
    contact_name VARCHAR(150),
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    country VARCHAR(100) NOT NULL DEFAULT 'Finland'
);
>
>
> ```

### 3.2 CREATE TABLE with Foreign Key

Write a CREATE TABLE statement for a `product_reviews` table:

- review_id (auto-incrementing primary key)
- product_id (required, references products)
- customer_id (required, references customers)
- rating (required integer, must be between 1 and 5 inclusive)
- review_text (optional, unlimited length)
- created_at (required, defaults to current timestamp)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- CREATE TABLE product_reviews (
    review_id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products(product_id) ON DELETE CASCADE,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id) ON DELETE CASCADE,
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    review_text TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
>
>
> ```

### 3.3 INSERT — Single Row

Write an INSERT statement to add a new category called 'Electronics' with description 'GPS devices, solar chargers, and tech gear'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>INSERT INTO categories (name, description)
VALUES ('Electronics', 'GPS devices, solar chargers, and tech gear');
>
> ```

### 3.4 INSERT — Multiple Rows

Write a single INSERT statement that adds three new customers:

- Eero Lahtinen, eero.l@email.com
- Maria Salminen, maria.s@email.com
- Petri Kallio, petri.k@email.com

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> --INSERT INTO customers (first_name, last_name, email) VALUES
('Eero', 'Lahtinen', 'eero.l@email.com'),
('Maria', 'Salminen', 'maria.s@email.com'),
('Petri', 'Kallio', 'petri.k@email.com');
>
>
> ```

### 3.5 INSERT with RETURNING

Write an INSERT statement that adds a new product called 'NorthStar GPS' priced at €229.99 with stock of 12, then assign it to category 'Electronics' (assume `category_id = 6`) using `product_categories`. Return the product_id and created_at.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- WITH new_product AS (
    INSERT INTO products (name, price, stock)
    VALUES ('NorthStar GPS', 229.99, 12)
    RETURNING product_id, created_at
),
link_category AS (
    INSERT INTO product_categories (product_id, category_id)
    SELECT product_id, 6 FROM new_product
)
SELECT product_id, created_at FROM new_product;
>
>
> ```

### 3.6 UPDATE — Simple

Write an UPDATE statement that changes the email of the customer with customer_id = 2 to 'mikko.korhonen@newmail.com'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- UPDATE customers
SET email = 'mikko.korhonen@newmail.com'
WHERE customer_id = 2;
>
>
> ```

### 3.7 UPDATE — Expression

Write an UPDATE statement that reduces the stock of all products by 1 where the stock is currently greater than 0.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- UPDATE products
SET stock = stock - 1
WHERE stock > 0;

>
>
> ```

### 3.8 UPDATE — Multiple Columns

Write an UPDATE statement that changes order #3 to status 'cancelled' and sets a (hypothetical) cancelled_at timestamp to the current time. (Assume you've already added a cancelled_at column.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> --ALTER TABLE orders ADD COLUMN cancelled_at TIMESTAMPTZ;

UPDATE orders
SET status = 'cancelled',
    cancelled_at = CURRENT_TIMESTAMP
WHERE order_id = 3
>
> ```

### 3.9 DELETE — With Condition

Write a DELETE statement that removes all orders with status 'cancelled'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>DELETE FROM orders
WHERE status = 'cancelled';

>
>
> ```

### 3.10 ALTER TABLE

Write the ALTER TABLE statements to:
a) Add a `discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100)` column to products
b) Drop the `description` column from categories
c) Add a composite unique constraint on (customer_id, product_id) in the product_reviews table (preventing a customer from reviewing the same product twice)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- a) Add discount_percent column
ALTER TABLE products 
ADD COLUMN discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100);

-- b) Drop description column
ALTER TABLE categories 
DROP COLUMN description;

-- c) Add composite unique constraint
ALTER TABLE product_reviews 
ADD CONSTRAINT uq_customer_product_review UNIQUE (customer_id, product_id);
>
>
> ```

---

## Exercise 4: Error Diagnosis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Each of the following SQL statements contains one or more errors. Identify the error(s) and write the corrected version.

### 4.1

```sql
CREATE TABLE warehouses
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Missing opening parenthesis ( after CREATE TABLE warehouses and missing closing parenthesis ) after city VARCHAR(100.)_

> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- CREATE TABLE warehouses (
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100)
);

>
>
> ```

### 4.2

```sql
INSERT INTO products (name, price, stock)
VALUES ("Alpine Sleeping Bag", 89.99, 20);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Used double quotes (") for string literals instead of single quotes (').)_

> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
>INSERT INTO products (name, price, stock)
VALUES ('Alpine Sleeping Bag', 89.99, 20);
>
>
> ```

### 4.3

```sql
CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id)
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Missing a comma , after order_id INTEGER REFERENCES orders(order_id).)_

> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
>
>
> ```

### 4.4

```sql
UPDATE products
SET price = price * 0.9
SET stock = stock + 10
WHERE product_id = 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Repeated SET keyword twice instead of separating assignments with a comma.)_

> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- UPDATE products
SET price = price * 0.9,
    stock = stock + 10
WHERE product_id = 3;
>
>
> ```

### 4.5

```sql
CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Defined two PRIMARY KEY constraints on the same table.)_

> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> --CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uq_customer_product_wishlist UNIQUE (customer_id, product_id)
);
>
>
> ```

---

## Submission Checklist

**Required**

- [ ] All 6 TrailShop tables created successfully
- [ ] Sample data inserted (at least 5 categories, 5 customers, 10 products, product_categories links, 5 orders, 10 order items)
- [ ] Theory review questions answered

**Recommended practice**

- [ ] UPDATE exercises completed and verified
- [ ] DELETE exercises completed and verified
- [ ] ALTER TABLE exercises completed, then `quantity_in_stock` renamed back to `stock`
- [ ] SQL writing exercises completed
- [ ] Error diagnosis completed with corrections
