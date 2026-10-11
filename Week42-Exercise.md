# Week 42 — Exercises: Review and Exam 1 Preparation

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Review and Verify

Before the exam, verify that your TrailShop database is complete and correct.

### Task 1.1: Schema Verification

Run the following checks and fix any issues:

1. Ran \dt - Confirm all 6 tables exist:categories, customers, products, product_categories, orders, order_items 
2. Ran \d categories`, `\d customers`, `\d products`, `\d product_categories`, `\d orders`, `\d order_items - all column definitions are correct
3. Verify all PRIMARY KEY constraints exist on each table.
4. Verify all FOREIGN KEY constraints exist and reference correctly.
5. Verify CHECK constraints on `price`, `stock`, `quantity`, `status` - all work correctly.
6. Verify NOT NULL constraints on all required columns.

### Task 1.2: Data Verification

1. SELECT COUNT(*) run on all tables — each returns > 0 rows
2.Join query ran successfully → returns matching data across all 6 tables, relationships work

```sql
SELECT c.first_name, cat.name AS category, p.name AS product,
       oi.quantity, o.status
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON oi.product_id = p.product_id
JOIN product_categories pc ON pc.product_id = p.product_id
JOIN categories cat ON cat.category_id = pc.category_id
ORDER BY c.first_name, o.order_id;
```

3. Insert into product_categories with invalid category_id → ERROR: foreign key constraint violated → fails as expected.
4. Update product price to -5 → ERROR: CHECK constraint violated → fails as expected

---

## Exercise 2: PetStore Mini-Case

This is a complete practice case similar to what you might see on the exam. Work through it from start to finish.

### 2.1 Business Description

**PetStore** is a small shop selling pet supplies. They need a database to track:
- **Product categories**: name (e.g., "Dog Food", "Cat Toys", "Fish Supplies")
- **Products**: name, description, price, stock quantity, which category they belong to
- **Customers**: first name, last name, email, phone
- **Sales**: which customer bought what product, how many, the price at time of sale, and the date

### 2.2 ER Diagram (Conceptual)

Draw or describe the ER diagram for PetStore:
- Entities: 
-pet _categories (category_id, name)
-pet _products  (product_id, name, description, price, stock, category_id)
-pet _customers (customer_id, first_name, last_name, email, phone), 
-pet _sales (sale_id, customer_id, product_id, quantity, unit_price, sale_date)
- Relationships:
    - `pet_categories` 1 → Many `pet_products` (one category has many products)
>   - `pet_customers` 1 → Many `pet_sales`
>   - `pet_products` 1 → Many `pet_sales`
>   - `pet_sales` links customers and products together

### 2.3 CREATE TABLE Statements

Write the CREATE TABLE statements:

```sql
CREATE TABLE pet_categories (
    category_id   SERIAL PRIMARY KEY,
    name          VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE pet_products (
    product_id    SERIAL PRIMARY KEY,
    name          VARCHAR(200) NOT NULL,
    description   TEXT,
    price         NUMERIC(8,2) NOT NULL CHECK (price > 0),
    stock         INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category_id   INTEGER NOT NULL REFERENCES pet_categories(category_id)
);

CREATE TABLE pet_customers (
    customer_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(100) NOT NULL,
    last_name     VARCHAR(100) NOT NULL,
    email         VARCHAR(255) NOT NULL UNIQUE,
    phone         VARCHAR(20)
);

CREATE TABLE pet_sales (
    sale_id       SERIAL PRIMARY KEY,
    customer_id   INTEGER NOT NULL REFERENCES pet_customers(customer_id),
    product_id    INTEGER NOT NULL REFERENCES pet_products(product_id),
    quantity      INTEGER NOT NULL CHECK (quantity > 0),
    unit_price    NUMERIC(8,2) NOT NULL CHECK (unit_price > 0),
    sale_date     TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 2.4 Sample Data

```sql
INSERT INTO pet_categories (name) VALUES
    ('Dog Food'), ('Cat Toys'), ('Fish Supplies'), ('Bird Cages'), ('Small Animal');

INSERT INTO pet_products (name, price, stock, category_id) VALUES
    ('Premium Kibble 10kg', 45.99, 30, 1),
    ('Grain-Free Wet Food 12-pack', 29.99, 50, 1),
    ('Laser Pointer', 12.99, 100, 2),
    ('Feather Wand', 8.99, 75, 2),
    ('Aquarium Filter 50L', 34.99, 20, 3),
    ('Tropical Fish Flakes', 6.99, 200, 3),
    ('Parakeet Cage Large', 89.99, 10, 4),
    ('Hamster Wheel', 14.99, 40, 5);

INSERT INTO pet_customers (first_name, last_name, email, phone) VALUES
    ('Liisa', 'Koivisto', 'liisa.k@email.com', '040-1234567'),
    ('Markus', 'Lehtinen', 'markus.l@email.com', NULL),
    ('Tiina', 'Järvinen', 'tiina.j@email.com', '050-9876543');

INSERT INTO pet_sales (customer_id, product_id, quantity, unit_price) VALUES
    (1, 1, 2, 45.99),
    (1, 3, 1, 12.99),
    (2, 5, 1, 34.99),
    (2, 6, 3, 6.99),
    (3, 4, 2, 8.99),
    (1, 7, 1, 89.99);
```

### 2.5 Practice Queries with Solutions

Write SQL for each, then check against the solution.

**Q1: List all products with their category names, sorted by category then price.**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT pc.name AS category, pp.name AS product, pp.price
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
ORDER BY pc.name, pp.price;
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pc.name AS category, pp.name AS product, pp.price
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
ORDER BY pc.name, pp.price;
```
</details>

**Q2: How much has each customer spent in total?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.first_name || ' ' || c.last_name AS customer,
       SUM(s.quantity * s.unit_price) AS total_spent
FROM pet_customers c
INNER JOIN pet_sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       SUM(s.quantity * s.unit_price) AS total_spent
FROM pet_customers c
INNER JOIN pet_sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```
</details>

**Q3: Which categories have products that have never been sold?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT DISTINCT pc.name AS category
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
WHERE s.sale_id IS NULL;
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT DISTINCT pc.name AS category
FROM pet_products pp
INNER JOIN pet_categories pc ON pp.category_id = pc.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
WHERE s.sale_id IS NULL;
```
</details>

**Q4: What is the total revenue per category?**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT pc.name AS category,
       COALESCE(SUM(s.quantity * s.unit_price), 0) AS revenue
FROM pet_categories pc
LEFT JOIN pet_products pp ON pc.category_id = pp.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pc.name
ORDER BY revenue DESC;
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pc.name AS category,
       COALESCE(SUM(s.quantity * s.unit_price), 0) AS revenue
FROM pet_categories pc
LEFT JOIN pet_products pp ON pc.category_id = pp.category_id
LEFT JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pc.name
ORDER BY revenue DESC;
```
</details>

**Q5: Find the most purchased product (by total quantity sold).**

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT pp.name, SUM(s.quantity) AS total_sold
FROM pet_products pp
INNER JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pp.product_id, pp.name
ORDER BY total_sold DESC
LIMIT 1;
>
>
> ```
>




<details>
<summary>Solution</summary>

```sql
SELECT pp.name, SUM(s.quantity) AS total_sold
FROM pet_products pp
INNER JOIN pet_sales s ON pp.product_id = s.product_id
GROUP BY pp.product_id, pp.name
ORDER BY total_sold DESC
LIMIT 1;
```
</details>

---

## Exercise 3: Additional Practice — Mixed Topics

These 10 questions cover material from Weeks 36–41. Write your answers without looking at notes first, then check.

### 3.1 (Week 36 — Theory)
Name three problems of file-based data management and explain how a database solves each one.

> [!NOTE]
> ***Your Answer***
>
>   Data Redundancy — same data stored in multiple files → waste space + inconsistent updates
    Database: single shared tables → store once, use many times
    Data Inconsistency — updates in one file not reflected in others
    Database: constraints + transactions → ensure all changes apply correctly
    Difficult Querying — searching/combining across files needs custom code every time
    Database: SQL → simple standard language to query/join data easily.
>
>
>
>

### 3.2 (Week 37 — Theory)
What is the difference between a candidate key and a primary key? Can a table have multiple candidate keys?

> [!NOTE]
> ***Your Answer***
>
> Candidate Key: any column/set of columns that uniquely identifies every row in a table
Primary Key: one chosen candidate key — officially designated unique identifier for the table
 Yes, a table can have multiple candidate keys but only one primary key
>
>
>
>

### 3.3 (Week 37 — Theory)
State the referential integrity rule. What happens in PostgreSQL if you try to INSERT a row that violates it?

> [!NOTE]
> ***Your Answer***
>
>Rule: If a table has a foreign key referencing another table, the value must either exist in the referenced table OR be NULL (if allowed). You cannot reference a non-existent record.
PostgreSQL behaviour: Immediately rejects the INSERT with a foreign key constraint violation error — no row is added.
>
>
>
>

### 3.4 (Week 38 — Design)
A library lends books to members. A member can borrow many books; a book can be borrowed by many members (over time). Draw or describe the ER model, including the junction entity.

> [!NOTE]
> ***Your Answer***
>
> Members (member_id, name, .)
Books (book_id, title, author, .)
Junction: Borrowings (borrowing_id, member_id, book_id, borrow_date, return_date)
Relationships: Members ↔ Borrowings (1:Many), Books ↔ Borrowings (1:Many)
Members and Books relate Many-to-Many through Borrowings
>
>
>
>

### 3.5 (Week 39 — Design)
For the library scenario above, write the CREATE TABLE statements (at least 3 tables with appropriate keys and constraints).

> [!NOTE]
> ***Your SQL***
>
> ```sql
>CREATE TABLE members (
    member_id   SERIAL PRIMARY KEY,
    first_name  VARCHAR(100) NOT NULL,
    last_name   VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE books (
    book_id     SERIAL PRIMARY KEY,
    title       VARCHAR(255) NOT NULL,
    author      VARCHAR(200) NOT NULL,
    isbn        CHAR(13) NOT NULL UNIQUE,
    publish_year INTEGER
);

CREATE TABLE borrowings (
    borrowing_id SERIAL PRIMARY KEY,
    member_id    INTEGER NOT NULL REFERENCES members(member_id),
    book_id      INTEGER NOT NULL REFERENCES books(book_id),
    borrow_date  DATE NOT NULL DEFAULT CURRENT_DATE,
    return_date  DATE
);
>
>
> ```

### 3.6 (Week 40 — DDL)
Write an ALTER TABLE statement that adds a `discount_percent NUMERIC(4,2) CHECK (discount_percent BETWEEN 0 AND 50)` column to a `products` table.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> ALTER TABLE products
ADD COLUMN discount_percent NUMERIC(4,2)
CHECK (discount_percent BETWEEN 0 AND 50);
>
>
> ```

### 3.7 (Week 40 — DML)
Write an INSERT statement that adds 3 rows to a `books` table with columns: title (VARCHAR), author (VARCHAR), isbn (CHAR(13)), price (NUMERIC). Make up realistic data.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO books (title, author, isbn, price) VALUES
    ('Database Systems: A Practical Approach', 'Sylvia Summers', '9780130402666', 54.90),
    ('Clean Code', 'Robert C. Martin', '9780132350884', 47.95),
    ('Introduction to Algorithms', 'Thomas H. Cormen', '9780262033848', 62.00);
>
>
> ```

### 3.8 (Week 41 — Querying)
Explain in plain English what this query returns:
```sql
SELECT c.name, COUNT(p.product_id)
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
GROUP BY c.name
HAVING COUNT(p.product_id) = 0;
```

> [!NOTE]
> ***Your Answer***
>
> This query returns category names that have zero products — categories with no products linked to them. It uses LEFT JOIN to include all categories, then filters where product count = 0.)*
>
>
>
>

### 3.9 (Week 41 — Querying)
Write a query that finds the top 3 customers by total spending, showing their full name and total amount spent.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT
    first_name || ' ' || last_name AS full_name,
    SUM(quantity * unit_price) AS total_spent
FROM pet_customers c
JOIN pet_sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, first_name, last_name
ORDER BY total_spent DESC
LIMIT 3;
>
>
> ```

### 3.10 (Week 41 — Querying)
Explain why this query is invalid:
```sql
SELECT pc.category_id, p.name, AVG(p.price)
FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
GROUP BY pc.category_id;
```

> [!NOTE]
> ***Your Answer***
>
> *(In PostgreSQL, every column in the SELECT clause that is not wrapped in an aggregate function (SUM, AVG, COUNT, etc.) must appear in the GROUP BY clause. Here p.name is selected but missing from GROUP BY pc.category_id → grouping cannot determine which product name to show per category.)*
>
>
>
>

---

## Exercise 4: Timed Practice Exercise (30 minutes)

Set a timer for 30 minutes. Complete as much as possible without looking at notes.

### Scenario: BookClub Database

A book club tracks its members, books, and reading sessions.

**Requirements:**
- Members: first_name, last_name, email (unique), join_date
- Books: title, author, isbn (unique, 13 chars), genre, page_count
- Reading sessions: which member read which book, start_date, end_date (nullable — still reading), rating (1-5, nullable — not yet rated)

### Tasks:

**Task A (5 min):** Write CREATE TABLE statements for all three tables with appropriate constraints.

**Task B (3 min):** Insert 3 members, 4 books, and 5 reading sessions.

**Task C (2 min):** Write an UPDATE that sets the end_date of member 1's reading of book 2 to today, and the rating to 4.

**Task D (5 min):** Write queries for:
1. All books that member 1 has read (show title, rating)
2. Average rating per book (only completed readings)
3. Members who have never rated any book below 3

**Task E (5 min):** Write a query showing: member name, number of books read, number of books currently reading (end_date IS NULL), average rating given.

**Task F (5 min):** Write a query finding books that NO member has read yet.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Paste your SQL for Tasks A–F here
>-- ========== TASK A: CREATE TABLES ==========
CREATE TABLE members (
    member_id   SERIAL PRIMARY KEY,
    first_name  VARCHAR(100) NOT NULL,
    last_name   VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    join_date   DATE NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE books (
    book_id     SERIAL PRIMARY KEY,
    title       VARCHAR(255) NOT NULL,
    author      VARCHAR(200) NOT NULL,
    isbn        CHAR(13) NOT NULL UNIQUE,
    genre       VARCHAR(100),
    page_count  INTEGER NOT NULL CHECK (page_count > 0)
);

CREATE TABLE reading_sessions (
    session_id  SERIAL PRIMARY KEY,
    member_id   INTEGER NOT NULL REFERENCES members(member_id),
    book_id     INTEGER NOT NULL REFERENCES books(book_id),
    start_date  DATE NOT NULL DEFAULT CURRENT_DATE,
    end_date    DATE,
    rating      INTEGER CHECK (rating BETWEEN 1 AND 5)
);

-- ========== TASK B: INSERT DATA ==========
INSERT INTO members (first_name, last_name, email) VALUES
    ('Anna', 'Virtanen', 'anna.v@example.com'),
    ('Jukka', 'Korhonen', 'jukka.k@example.com'),
    ('Sari', 'Niemi', 'sari.n@example.com');

INSERT INTO books (title, author, isbn, genre, page_count) VALUES
    ('The Art of SQL', 'Stephane Faroult', '9780596008947', 'Database', 384),
    ('SQL Performance Explained', 'Markus Winand', '9783950307827', 'Database', 248),
    ('Designing Data-Intensive Apps', 'Martin Kleppmann', '9781491903100', 'Systems', 590),
    ('Database Design for Mere Mortals', 'Michael Hernandez', '9780321884497', 'Design', 650);

INSERT INTO reading_sessions (member_id, book_id, start_date, end_date, rating) VALUES
    (1, 1, '2026-09-01', '2026-09-15', 5),
    (1, 2, '2026-09-16', NULL, NULL),
    (2, 1, '2026-09-05', '2026-09-20', 4),
    (2, 3, '2026-10-01', '2026-10-10', 5),
    (3, 4, '2026-10-05', NULL, NULL);

-- ========== TASK C: UPDATE ==========
UPDATE reading_sessions
SET end_date = CURRENT_DATE,
    rating = 4
WHERE member_id = 1
  AND book_id = 2;

-- ========== TASK D: QUERIES ==========
-- D1: All books member 1 has read
SELECT b.title, rs.rating
FROM reading_sessions rs
JOIN books b ON rs.book_id = b.book_id
WHERE rs.member_id = 1;

-- D2: Average rating per book (completed only)
SELECT
    b.title,
    ROUND(AVG(rs.rating), 2) AS avg_rating
FROM books b
JOIN reading_sessions rs ON b.book_id = rs.book_id
WHERE rs.end_date IS NOT NULL
GROUP BY b.book_id, b.title;

-- D3: Members never rated any book below 3
SELECT DISTINCT m.first_name || ' ' || m.last_name AS member_name
FROM members m
WHERE NOT EXISTS (
    SELECT 1 FROM reading_sessions rs
    WHERE rs.member_id = m.member_id
      AND rs.rating IS NOT NULL
      AND rs.rating < 3
);

-- ========== TASK E: Summary Report ==========
SELECT
    m.first_name || ' ' || m.last_name AS member_name,
    COUNT(CASE WHEN rs.end_date IS NOT NULL THEN 1 END) AS books_completed,
    COUNT(CASE WHEN rs.end_date IS NULL THEN 1 END) AS books_reading,
    ROUND(AVG(rs.rating), 2) AS avg_rating_given
FROM members m
LEFT JOIN reading_sessions rs ON m.member_id = rs.member_id
GROUP BY m.member_id, member_name;

-- ========== TASK F: Books no one has read ==========
SELECT b.title, b.author
FROM books b
LEFT JOIN reading_sessions rs ON b.book_id = rs.book_id
WHERE rs.session_id IS NULL;
>
> ```

**Task G (5 min):** Write ALTER TABLE statements to:
1. Add a `pages_read INTEGER DEFAULT 0` column to reading sessions
2. Add a CHECK constraint ensuring pages_read <= the book's page_count (Hint: this requires a trigger or application logic — explain why a simple CHECK can't do cross-table validation)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- G1: Add pages_read column
ALTER TABLE reading_sessions
ADD COLUMN pages_read INTEGER NOT NULL DEFAULT 0;

-- G2: Note: direct CHECK constraint NOT possible — see explanation below
>
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(A simple CHECK(pages_read <= page_count) cannot work because pages_read lives in reading_sessions while page_count lives in the separate books table. Standard CHECK constraints only validate values within the same row/table. Cross-table validation requires:
A trigger function that joins to books and validates before insert/update, OR
Application-level validation in your code)*
>
>
>
>

---

## Submission Checklist

- [ ] TrailShop database verified (schema + data + constraints tested)
- [ ] PetStore mini-case completed (tables created, data inserted, all 5 queries working)
- [ ] All 10 mixed-topic questions answered
- [ ] Timed practice completed (note how much you finished in 30 minutes)
- [ ] Self-assessment checklist from Theory filled in — study weak areas before the exam
