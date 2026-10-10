# Week 41 — Exercises: Querying Data

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Business Questions

Use the TrailShop database you created in Week 40. Write SQL queries to answer each business question below. Run each query and verify the results make sense.

> [!IMPORTANT]
> **_Tools to use_**
> You can use both the pgAdmin or psql (terminal) to test the queries

### Basic Queries (SELECT + WHERE)

1. List all products in the 'Footwear' category (show name, price, stock).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT p.name, p.price, p.stock
   > FROM products p
   > JOIN product_categories pc ON p.product_id = pc.product_id
   > JOIN categories c ON pc.category_id = c.category_id
   > WHERE c.name = 'Footwear';
   >
   >
   > ```

2. Find all products priced between €50 and €150, sorted by price ascending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT name, price, stock
   > FROM products
   > WHERE price BETWEEN 50 AND 150
   > ORDER BY price ASC;
   >
   >
   > ```

3. Show all customers whose last name starts with the letter 'M' or 'K'.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT first_name, last_name, email
   > FROM customers
   > WHERE last_name LIKE 'M%' OR last_name LIKE 'K%';
   >
   >
   > ```

4. List all orders with status 'pending' or 'shipped', sorted by order date (most recent first).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT order_id, customer_id, order_date, status
   > FROM orders
   > WHERE status IN ('pending', 'shipped')
   > ORDER BY order_date DESC;
   >
   >
   > ```

5. Find all products that have the word 'Pro' or 'pro' somewhere in their name.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT name, price
   > FROM products
   > WHERE LOWER(name) LIKE '%pro%';
   >
   >
   > ```

### Aggregate Queries

6. What is the total number of products in the database?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT COUNT(*) AS total_products
> FROM products;
>
>
> ```

7. What is the average price of all products? Round to 2 decimal places.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT ROUND(AVG(price), 2) AS average_price
   > FROM products;
   >
   >
   > ```

8. Which is the most expensive product and which is the cheapest? Show both in one query.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > (SELECT name, price, 'Most Expensive' AS price_rank
   >  FROM products
   >  WHERE price = (SELECT MAX(price) FROM products))
   > UNION ALL
   > (SELECT name, price, 'Cheapest' AS price_rank
   >  FROM products
   > WHERE price = (SELECT MIN(price) FROM products));
   >
   >
   > ```

9. How many orders does each customer have? Show customer name and order count, sorted by count descending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
       COUNT(o.order_id) AS order_count
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
    ORDER BY order_count DESC;
   >
   > ```

11. What is the total revenue (sum of quantity × unit_price from order_items) for each order status?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>SELECT o.status,
       SUM(oi.quantity * oi.unit_price) AS total_revenue
       FROM orders o
       JOIN order_items oi ON o.order_id = oi.order_id
       GROUP BY o.status;
>
>
> ```

### JOIN Queries

11. List all products with their category names (join through `product_categories`). A product in two categories should appear twice. Sort by category name, then product name.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.name AS category_name, p.name AS product_name
    > FROM categories c
    > JOIN product_categories pc ON c.category_id = pc.category_id
    > JOIN products p ON pc.product_id = p.product_id
    > ORDER BY category_name, product_name;
    >
    >
    > ```

12. Show each order with the customer's full name, order date, and status.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    >SELECT o.order_id,
    > CONCAT(c.first_name, ' ', c.last_name) AS customer_full_name,
    >  o.order_date,
    > o.status
    > FROM orders o
    > JOIN customers c ON o.customer_id = c.customer_id;
    >
    >
    > ```

13. Show a detailed breakdown of order #1: product name, quantity, unit price, and line total.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT p.name AS product_name,
    > oi.quantity,
    > oi.unit_price,
    > oi.quantity * oi.unit_price AS line_total
    > FROM orders o
    > JOIN order_items oi ON o.order_id = oi.order_id
    > JOIN products p ON oi.product_id = p.product_id
    > WHERE o.order_id = 1;
    >
    >
    > ```

14. Find all customers who have NOT placed any orders. (Hint: use LEFT JOIN + IS NULL pattern.)

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.customer_id,
    > CONCAT(c.first_name, ' ', c.last_name) AS customer_name
    > FROM customers c
    > LEFT JOIN orders o ON c.customer_id = o.customer_id
    > WHERE o.order_id IS NULL;
    >
    >
    > ```

15. For each category, show the category name, number of products, average price, and total inventory value (price × stock summed). Only include categories with total inventory value greater than €500.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.name AS category_name,
    > COUNT(DISTINCT p.product_id) AS product_count,
    > ROUND(AVG(p.price), 2) AS average_price,
    >  SUM(p.price * p.stock) AS total_inventory_value
    > FROM categories c
    > JOIN product_categories pc ON c.category_id = pc.category_id
    > JOIN products p ON pc.product_id = p.product_id
    > GROUP BY c.category_id, c.name
    > HAVING SUM(p.price * p.stock) > 500
    > ORDER BY total_inventory_value DESC;
    >
    >
    > ```

---

## Exercise 2: Theory Review Questions

Answer in your own words:

1. What is the logical execution order of a SQL query? Why does it matter?

> [!NOTE]
> **_Your Answer_**
>
> _(FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT.
It determines when filters/aliases are available. For example, WHERE runs before SELECT so you can’t use a column alias there; understanding this prevents errors.)_

2. What is the difference between WHERE and HAVING? Give an example of when you would use each.

> [!NOTE]
> **_Your Answer_**
>
> _(WHERE filters rows before grouping; HAVING filters groups after GROUP BY.
Example: WHERE price > 100 filters individual products; HAVING COUNT(*) > 5 filters categories by product count.)_

3. Explain the difference between COUNT(\*), COUNT(column), and COUNT(DISTINCT column).

> [!NOTE]
> **_Your Answer_**
>
> _(COUNT(*) — counts all rows, including NULLs
COUNT(col) — counts non‑NULL values only
COUNT(DISTINCT col) — counts unique non‑NULL values.)_

4. What is the difference between INNER JOIN and LEFT JOIN? When would you choose one over the other?

> [!NOTE]
> **_Your Answer_**
>
> _(INNER JOIN returns only matching rows from both tables. LEFT JOIN keeps all rows from the left table, filling missing matches with NULL. Use INNER JOIN for linked data only; use LEFT JOIN to keep all records from one side.)_

5. Why should you avoid SELECT \* in production code?

> [!NOTE]
> **_Your Answer_**
>
> _(Pulls unnecessary data, wastes resources. Schema changes break results. Explicit columns make code clearer and safer.)_

6. What does DISTINCT do? On what level does it operate (columns or entire rows)?

> [!NOTE]
> **_Your Answer_**
>
> _(Removes duplicate rows. It works on entire rows — all selected columns together determine uniqueness.)_

7. Can you use a column alias in a WHERE clause? Why or why not?

> [!NOTE]
> **_Your Answer_**
>
> _(No. WHERE runs before SELECT, so the alias doesn’t exist yet. Use the original column/expression instead.)_
>
> 8. Explain what happens when you GROUP BY a column and there's a column in SELECT that isn't aggregated and isn't in GROUP BY.

> [!NOTE]
> **_Your Answer_**
>
> _(This causes an error. Every column in SELECT must be in GROUP BY or wrapped in an aggregate — otherwise the database doesn’t know which value to return per group.)_

---

## Exercise 3: Query Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL for each task. Use the TrailShop schema (categories, customers, products, product_categories, orders, order_items).

### Simple SELECT + WHERE

**3.1** — Select product names and prices for all products with stock less than 15.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT name, price
> FROM products
> WHERE stock < 15;
>
>
> ```

**3.2** — Find all customers who registered (created_at) in 2026. Show first name, last name, and registration date.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>SELECT first_name, last_name, created_at AS registration_date
> FROM customers
> WHERE EXTRACT(YEAR FROM created_at) = 2026;
>
>
> ```

**3.3** — Show all products that do NOT have a description (description IS NULL).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>SSELECT name, product_id
> FROM products
> WHERE description IS NULL;
>
> ```

### Multi-condition Filtering

**3.4** — Find products in category 1 OR category 2, priced above €100, with stock greater than 0. Sort by price descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT DISTINCT p.name, p.price
> FROM products p
> JOIN product_categories pc ON p.product_id = pc.product_id
> WHERE pc.category_id IN (1, 2)
> AND p.price > 100
> AND p.stock > 0
> ORDER BY p.price DESC;
>
>
> ```

**3.5** — Find orders that are either 'delivered' or placed by customer_id 1. Show order_id, customer_id, status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>SELECT order_id, customer_id, status
> FROM orders
> WHERE status = 'delivered' OR customer_id = 1;
>
>
> ```

### Aggregation

**3.6** — For each category, show the category name, minimum, maximum, and average product price. Round averages to 2 decimal places. Join through `product_categories`.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT c.name AS category_name,
       MIN(p.price) AS min_price,
       MAX(p.price) AS max_price,
       ROUND(AVG(p.price), 2) AS avg_price
       FROM categories c
       JOIN product_categories pc ON c.category_id = pc.category_id
       JOIN products p ON pc.product_id = p.product_id
       GROUP BY c.category_id, c.name;
>
>
> ```

**3.7** — Count how many distinct customers have placed at least one order.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
>SELECT COUNT(DISTINCT customer_id) AS distinct_customer_count
> FROM orders;
>
>
> ```

**3.8** — Find the total quantity of items sold across all orders.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT SUM(quantity) AS total_items_sold
> FROM order_items;
>
>
> ```

### JOIN Queries

**3.9** — Show each product name alongside its category name. Include all products (even if somehow a category was deleted — use LEFT JOIN).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT p.name AS product_name,
       c.name AS category_name
       FROM products p
       LEFT JOIN product_categories pc ON p.product_id = pc.product_id
       LEFT JOIN categories c ON pc.category_id = c.category_id;
>
>
> ```

**3.10** — List all orders showing: order_id, customer full name, order date, number of items in the order, and order total cost.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT o.order_id,
       CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
       o.order_date,
       COUNT(DISTINCT oi.product_id) AS item_count,
       SUM(oi.quantity * oi.unit_price) AS order_total
       FROM orders o
       JOIN customers c ON o.customer_id = c.customer_id
       LEFT JOIN order_items oi ON o.order_id = oi.order_id
       GROUP BY o.order_id, c.first_name, c.last_name;
>
>
> ```

**3.11** — Show all products that have NEVER been ordered. (Hint: LEFT JOIN order_items, then IS NULL on order_item_id.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT p.product_id, p.name
> FROM products p
> LEFT JOIN order_items oi ON p.product_id = oi.product_id
> WHERE oi.order_item_id IS NULL;
>
>
> ```

### GROUP BY + HAVING

**3.12** — Show categories where the average product price exceeds €100. Display category name and average price.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT c.name AS category_name,
       ROUND(AVG(p.price), 2) AS average_price
       FROM categories c
       JOIN product_categories pc ON c.category_id = pc.category_id
       JOIN products p ON pc.product_id = p.product_id
       GROUP BY c.category_id, c.name
       HAVING AVG(p.price) > 100;
>
>
> ```

**3.13** — Find customers who have placed more than 1 order. Show their name and order count.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
       COUNT(o.order_id) AS order_count
       FROM customers c
       JOIN orders o ON c.customer_id = o.customer_id
       GROUP BY c.customer_id, c.first_name, c.last_name
       HAVING COUNT(o.order_id) > 1;
>
>
> ```

### Pagination

**3.14** — Write a paginated query that returns products 4 through 6 (page 2, page size 3), ordered by product_id.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT *
> FROM products
> ORDER BY product_id
> OFFSET 3 LIMIT 3;
>
>
> ```

### Complex

**3.15** — Write a "sales report" query that shows: category name, total units sold (from order_items), total revenue, and number of distinct products sold — for each category. Sort by revenue descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT c.name AS category_name,
       SUM(oi.quantity) AS total_units_sold,
       SUM(oi.quantity * oi.unit_price) AS total_revenue,
       COUNT(DISTINCT oi.product_id) AS products_sold_count
       FROM categories c
       JOIN product_categories pc ON c.category_id = pc.category_id
       JOIN order_items oi ON pc.product_id = oi.product_id
       GROUP BY c.category_id, c.name
       ORDER BY total_revenue DESC;
>
>
> ```

---

## Exercise 4: Query Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, explain in **plain English** what it does and what the result would look like.

### 4.1

```sql
SELECT c.name, COUNT(p.product_id) AS num_products
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
GROUP BY c.name
ORDER BY num_products DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(COUNT(p.product_id) works but LEFT JOIN can overcount products in multiple categories.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> SELECT c.name, COUNT(DISTINCT p.product_id) AS num_products
> FROM categories c
> LEFT JOIN product_categories pc ON pc.category_id = c.category_id
> LEFT JOIN products p ON p.product_id = pc.product_id
> GROUP BY c.name
> ORDER BY num_products DESC;
>
>
> ```

### 4.2

```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(No syntax error, but if orders.customer_id can be NULL, NOT IN fails. Safer with LEFT JOIN/IS NULL.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> SELECT c.first_name, c.last_name
> FROM customers c
> LEFT JOIN orders o ON c.customer_id = o.customer_id
> WHERE o.order_id IS NULL;
>
>
> ```

### 4.3

```sql
SELECT p.name, p.price, p.stock,
       p.price * p.stock AS inventory_value
FROM products p
WHERE p.stock > 0
ORDER BY inventory_value DESC
LIMIT 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(No error — logic is correct. If instructor insists on qualification, table alias clarity added below.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> SELECT p.name, p.price, p.stock,
       p.price * p.stock AS inventory_value
       FROM products p
       WHERE p.stock > 0
       ORDER BY inventory_value DESC
       LIMIT 3;
>
>
> ```

### 4.4

```sql
SELECT o.order_id,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status <> 'cancelled'
GROUP BY o.order_id
HAVING SUM(oi.quantity * oi.unit_price) > 200
ORDER BY order_total DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _( Logically fine; repeating the SUM expression is verbose but not wrong. Can simplify with CTE.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> SELECT o.order_id,
       SUM(oi.quantity * oi.unit_price) AS order_total
       FROM orders o
       INNER JOIN order_items oi ON o.order_id = oi.order_id
       WHERE o.status <> 'cancelled'
       GROUP BY o.order_id
       HAVING SUM(oi.quantity * oi.unit_price) > 200
       ORDER BY order_total DESC;
>
>
> ```

### 4.5

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       COUNT(DISTINCT o.order_id) AS num_orders,
       SUM(oi.quantity) AS total_items,
       ROUND(SUM(oi.quantity * oi.unit_price), 2) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC NULLS LAST;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(GROUP BY includes c.first_name, c.last_name alongside c.customer_id — functionally fine but redundant since customer_id is unique. Not strictly wrong, simplified below.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> SELECT c.first_name || ' ' || c.last_name AS customer,
       COUNT(DISTINCT o.order_id) AS num_orders,
       COALESCE(SUM(oi.quantity), 0) AS total_items,
       COALESCE(ROUND(SUM(oi.quantity * oi.unit_price), 2), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC NULLS LAST;
>
>
> ```

---

## Submission Checklist

**Required**

- [ ] All 15 business questions answered with working SQL
- [ ] Theory review questions answered in your own words
- [ ] Results of the business questions verified by running them against your TrailShop database

**Recommended practice**

- [ ] All 15 query writing exercises completed
- [ ] All 5 query reading explanations written
