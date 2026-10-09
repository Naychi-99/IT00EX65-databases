# Week 39 — Logical Database Design: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 39 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Build the Schema

**Goal:** Convert the TrailShop ER diagram (from Week 38) into a complete PostgreSQL relational schema.

### Instructions

Write `CREATE TABLE` statements for all six TrailShop tables:

1. `categories`
2. `customers`
3. `products`
4. `product_categories`
5. `orders`
6. `order_items`

### Requirements

For each table, you must:

- Choose appropriate PostgreSQL data types for every column (justify at least 3 choices in writing)
- Define primary keys (surrogate or composite as appropriate)
- Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions (justify each choice)
- Add `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints where appropriate
- Create tables in the correct dependency order
- Follow the naming conventions from Theory Section 8

### Deliverables

1. A single `.sql` file with all six `CREATE TABLE` statements (executable in PostgreSQL)
2. A short justification for data types, FK actions and design decisions:
   - Justification for 3 data type choices (e.g., why `NUMERIC(10,2)` for price instead of `REAL`)
   - Justification for each FK action choice (e.g., why CASCADE on `order_items.order_id`)
   - One design decision you made that wasn't specified in the requirements (e.g., whether shipping address is optional)

### Bonus Challenge

After creating the tables, insert sample data:
- At least 5 categories
- At least 8 products (across at least 3 categories)
- At least one product assigned to **two or more** categories via `product_categories`
- At least 3 customers
- At least 4 orders (across at least 2 customers)
- At least 10 order items

Verify that your constraints work by attempting at least 2 invalid inserts and showing the error messages.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Paste key CREATE TABLE sta-============================================================================
-- EXERCISE 1: TrailShop Project Schema
-- ============================================================================

CREATE TABLE categories (
    category_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);

CREATE TABLE customers (
    customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INT NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE product_categories (
    product_id INT NOT NULL REFERENCES products(product_id) ON DELETE CASCADE ON UPDATE CASCADE,
    category_id INT NOT NULL REFERENCES categories(category_id) ON DELETE RESTRICT ON UPDATE CASCADE,
    PRIMARY KEY (product_id, category_id)
);

CREATE TABLE orders (
    order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(customer_id) ON DELETE RESTRICT ON UPDATE CASCADE,
    order_date TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'new' CHECK (status IN ('new', 'processing', 'shipped', 'delivered', 'cancelled')),
    shipping_address TEXT NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL DEFAULT 0.00 CHECK (total_amount >= 0)
);

CREATE TABLE order_items (
    order_id INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE ON UPDATE CASCADE,
    product_id INT NOT NULL REFERENCES products(product_id) ON DELETE RESTRICT ON UPDATE CASCADE,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY (order_id, product_id)
);
>
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Data Types:
> Numeric -exact decimal values,no floating-point errors
> TIMESTAMPTZ - stores UTC + auto timezone conversion, no ambiguity
> INTEGER - small, fast, perfect for whole-number counts.
> FK Actions:
> CASCADE for order_items & product_categories - delete together when parent is gone
> RESTRICT for orders.customer_id & order_items.product_id - block delete if history
> exists.
> Extra Decision: Added CHECK for status values & DEFAULT CURRENT_TIMESTAMP for audit.)*
>
>
>
>

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. List the seven phases of the database development lifecycle in order. Which phase is this week's focus? *(Section 1)*

> [!NOTE]
> ***Your Answer***
>
> *(The seven phases are Requirements Analysis, Conceptual Design, Logical Design, Schema
> Refinement (Normalization), Physical Design, Implementation, and Maintenance; this
> week's focus is Logical Database Design..)*
>
>
>
>

2. Explain the transformation rule for mapping a 1:N relationship to the relational model. Why is the foreign key placed on the "many" side? *(Section 3.2)*

> [!NOTE]
> ***Your Answer***
>
> *(Put FK in the "many" table. Each child has exactly one parent, so one FK field links
> all children efficiently..)*
>
>
>
>

3. What is a junction table? When is it needed? Give an example not from TrailShop. *(Section 3.3)*

> [!NOTE]
> ***Your Answer***
>
> *(Junction table = connects two M:N tables. Needed when both sides can relate to many →
> e.g., student_course links students ↔ courses.)*
>
>
>
>

4. When mapping a 1:1 relationship, how do you decide which table gets the foreign key? *(Section 3.4)*

> [!NOTE]
> ***Your Answer***
>
> *(Put FK on the side where the relationship is mandatory or where it's more frequently
> accessed. Add UNIQUE to enforce 1:1..)*
>
>
>
>

5. How does the mapping of a weak entity differ from a strong entity? What happens to the primary key? *(Section 3.5)*

> [!NOTE]
> ***Your Answer***
>
> *(Weak entity has no independent PK — depends on another. Its PK = parent PK + its own
> partial key.)*
>
>
>
>

6. Why should you never use `REAL` or `DOUBLE PRECISION` for monetary values? What should you use instead? *(Section 4.1)*

> [!NOTE]
> ***Your Answer***
>
> *( Floating-point types store approximate values and introduce rounding errors that accumulate during calculations — unacceptable for money. Always use `NUMERIC` or `DECIMAL` types, which store exact decimal values..)*
>
>
>
>

7. What is the difference between `TIMESTAMP` and `TIMESTAMPTZ`? Which should you prefer and why? *(Section 4.3)*
> [!NOTE]
> ***Your Answer***
>
> *( TIMESTAMP = no timezone info. TIMESTAMPTZ = UTC internally + converts on display. Use
> TIMESTAMPTZ always)*
>




8. Explain the difference between `CASCADE` and `RESTRICT` as foreign key delete actions. Give a scenario where each is appropriate. *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> *(CASCADE = delete/update children automatically. RESTRICT = block if children exist.
> Use CASCADE for dependent rows, RESTRICT to protect history.)*
>




9. What is an insertion anomaly? Give an example and explain how proper schema design prevents it. *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(Can't add one entity without adding another. Fix: proper normalization → separate
> tables.)*
>




10. What is the difference between a surrogate key and a natural key? Give one advantage of each. *(Section 9)*
> [!NOTE]
> ***Your Answer***
>
> *( Natural key = real-world data (e.g. email) → meaningful. Surrogate key = system
> generated ID → never changes.)*
>




11. Why does PostgreSQL fold unquoted identifiers to lowercase? How does `snake_case` naming help? *(Section 8)*

> [!NOTE]
> ***Your Answer***
>
> *( PostgreSQL auto-lowercases unquoted names. snake_case avoids quotes & keeps it
> portable..)*
>
>
>
>

12. What does `SET NULL` do as a foreign key action? When would you use it instead of `CASCADE`? *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> *(SET NULL → set FK to NULL instead of deleting rows. Use when relationship is optional
> (e.g. employee without a project)..)*
>




---

## Exercise 3: Transformation Exercise — Hotel Booking System

### Given ER Diagram

A hotel booking system has the following entities and relationships:

**Entities:**

1. **Hotel** — hotel_id (PK), name, city, star_rating, phone
2. **Room** (weak entity, owned by Hotel) — room_number (partial key), room_type, floor, price_per_night, has_balcony
3. **Guest** — guest_id (PK), first_name, last_name, email, phone, passport_number
4. **Booking** — booking_id (PK), check_in_date, check_out_date, total_amount, status
5. **Service** — service_id (PK), name, description, price (e.g., "Room Service", "Spa", "Airport Shuttle")

**Relationships:**

- Hotel (1) → Room (N): A hotel has many rooms. Each room belongs to exactly one hotel. (Identifying relationship — Room is weak.)
- Guest (1) → Booking (N): A guest can make many bookings. Each booking belongs to one guest.
- Booking (M) ↔ Room (N): A booking can include multiple rooms, and a room can appear in many bookings (over time). The junction records the specific dates.
- Booking (M) ↔ Service (N): A booking can use multiple services, and a service can be used by many bookings. The junction records the date used and quantity.

### Task

1. Write `CREATE TABLE` statements for ALL tables (including junction tables).
2. For each table:
   - Choose appropriate data types
   - Define PK, FK, NOT NULL, UNIQUE, CHECK, and DEFAULT constraints
   - Specify ON DELETE and ON UPDATE actions for all FKs
3. Create the tables in the correct dependency order.
4. Explain why Room is a weak entity and how its PK reflects this.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your CREATE TABLE statements he-- ============================================================================
>DROP TABLE IF EXISTS booking_services CASCADE;
DROP TABLE IF EXISTS booking_rooms CASCADE;
DROP TABLE IF EXISTS services CASCADE;
DROP TABLE IF EXISTS bookings CASCADE;
DROP TABLE IF EXISTS guests CASCADE;
DROP TABLE IF EXISTS rooms CASCADE;
DROP TABLE IF EXISTS hotels CASCADE;

-- ============================================================================
-- EXERCISE 3: Hotel Booking System Schema
-- ============================================================================

CREATE TABLE hotels (
    hotel_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    city VARCHAR(100) NOT NULL,
    star_rating INT CHECK (star_rating BETWEEN 1 AND 5),
    phone VARCHAR(30) NOT NULL
);

CREATE TABLE rooms (
    hotel_id INT NOT NULL REFERENCES hotels(hotel_id) ON DELETE CASCADE ON UPDATE CASCADE,
    room_number VARCHAR(10) NOT NULL,
    room_type VARCHAR(50) NOT NULL,
    floor INT NOT NULL CHECK (floor >= 0),
    price_per_night NUMERIC(10, 2) NOT NULL CHECK (price_per_night >= 0),
    has_balcony BOOLEAN NOT NULL DEFAULT FALSE,
    PRIMARY KEY (hotel_id, room_number)
);

CREATE TABLE guests (
    guest_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(30) NOT NULL,
    passport_number VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE bookings (
    booking_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    guest_id INT NOT NULL REFERENCES guests(guest_id) ON DELETE RESTRICT ON UPDATE CASCADE,
    check_in_date DATE NOT NULL,
    check_out_date DATE NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'confirmed' CHECK (status IN ('confirmed', 'cancelled', 'checked_in', 'completed')),
    CHECK (check_out_date > check_in_date)
);

CREATE TABLE services (
    service_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0)
);

CREATE TABLE booking_rooms (
    booking_id INT NOT NULL REFERENCES bookings(booking_id) ON DELETE CASCADE ON UPDATE CASCADE,
    hotel_id INT NOT NULL,
    room_number VARCHAR(10) NOT NULL,
    PRIMARY KEY (booking_id, hotel_id, room_number),
    FOREIGN KEY (hotel_id, room_number) REFERENCES rooms(hotel_id, room_number) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE booking_services (
    booking_id INT NOT NULL REFERENCES bookings(booking_id) ON DELETE CASCADE ON UPDATE CASCADE,
    service_id INT NOT NULL REFERENCES services(service_id) ON DELETE RESTRICT ON UPDATE CASCADE,
    service_date DATE NOT NULL DEFAULT CURRENT_DATE,
    quantity INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
    PRIMARY KEY (booking_id, service_id, service_date)
);
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Room is a weak entity because a room number (like "101") is not globally unique on its
>  own and cannot exist independently of a specific hotel. Its primary key is a composite
> key composed of the parent hotel's foreign key (hotel_id) combined with its local
> discriminator (room_number).)*
>
>
>
>

---

## Exercise 4: Data Type Selection

> [!NOTE]
> ***Your Answers***
> Fill in the **Your Data Type** and **Justification** columns in the table below.
>

For each column described below, choose the best PostgreSQL data type and write a brief justification (1–2 sentences). Do NOT just pick `VARCHAR` or `TEXT` for everything — think carefully about validation, storage, and query needs.

| # | Column Description | Your Data Type | Justification |
|---|---|---|---|
| 1 | Employee salary (exact, up to €999,999.99) | NUMERIC(10,2)|Stores exact decimal values; avoids floating-point rounding errors — essential for currency. |
| 2 | Number of items in stock (never negative, max ~50,000) |	INTEGER | Efficient whole-number storage; easily covers the range; add CHECK (col >= 0) to prevent negatives.|
| 3 | Whether a user's email is verified | 	BOOLEAN |Native true/false type; compact and clearly expresses a binary state. |
| 4 | Customer's date of birth | DATE | Stores only date without time; ideal for birthdays and validates calendar values automatically. |
| 5 | Product description (variable length, could be several paragraphs) | TEXT | Unlimited length; efficient storage with no arbitrary character limit. |
| 6 | Country code (always exactly 2 letters, like "FI", "US") | 	CHAR(2) | Fixed-length type enforces exactly 2 characters; matches ISO country code format. |
| 7 | IP address of a login attempt | INET | PostgreSQL native type; validates IP format and supports network-based queries. |
| 8 | Order total (exact, up to €9,999,999.99) | NUMERIC(12,2) | Exact decimal arithmetic; sufficient precision and range for large monetary values. |
| 9 | GPS latitude of a store location | NUMERIC(9,6) | Preserves sub-meter precision; avoids rounding drift common with floating-point types.|
| 10 | A unique identifier for API tokens that must be globally unique across distributed systems | UUID | Standard 128-bit format; designed for collision-free global identification.|
| 11 | Duration of a video in seconds (always a whole number) |INTEGER | Compact and fast for whole numbers; no fractional values needed.|
| 12 | Timestamp of when a record was last modified (users in multiple time zones) |TIMESTAMPTZ | Stores in UTC internally and converts on display; removes timezone ambiguity.|
| 13 | A Finnish phone number like "+358 40 123 4567" |VARCHAR(30) | Preserves formatting, spaces, and leading characters; phone numbers are not mathematically operated on. |
| 14 | A percentage discount (0.00% to 100.00%) | NUMERIC(5,2) | Exact decimal percentages; add CHECK (col BETWEEN 0 AND 100) to enforce valid range.|
| 15 | A product's color options (e.g., a product comes in "red", "blue", "green") | VARCHAR(20) or ENUM| ENUM restricts to allowed values directly; VARCHAR offers flexibility if options expand later.|

---

## Exercise 5: Constraint Design

For each business rule below, write the appropriate PostgreSQL constraint. Provide the constraint as it would appear inside a `CREATE TABLE` statement or as an `ALTER TABLE` statement.

### Part A: Single-Column Constraints

1. "A product's weight must be greater than zero (if provided)."

2. "Every customer must have an email address."

3. "Product names must be unique."

4. "An employee's hire date defaults to today if not specified."

5. "Order status can only be one of: 'new', 'confirmed', 'shipped', 'delivered', 'returned'."

> [!NOTE]
> ***Your SQL***
>
> ```sql
> ALTER TABLE products ADD CONSTRAINT chk_weight_positive CHECK (weight IS NULL OR weight > 0);
ALTER TABLE customers ALTER COLUMN email SET NOT NULL;
ALTER TABLE products ADD CONSTRAINT uq_product_name UNIQUE (name);
ALTER TABLE employees ALTER COLUMN hire_date SET DEFAULT CURRENT_DATE;
ALTER TABLE orders ADD CONSTRAINT chk_order_status CHECK (status IN ('new', 'confirmed', 'shipped', 'delivered', 'returned'));
>
>
> ```

### Part B: Multi-Column Constraints

6. "A flight's arrival time must be after its departure time."

7. "In the `enrollments` table, the combination of `student_id` and `course_id` must be unique (a student can only enroll in a course once)."

8. "A discount percentage must be between 0 and 100, inclusive."

> [!NOTE]
> ***Your SQL***
>
> ```sql
> ALTER TABLE flights ADD CONSTRAINT chk_arrival_after_departure CHECK (arrival_time > departure_time);
ALTER TABLE enrollments ADD CONSTRAINT uq_student_course UNIQUE (student_id, course_id);
ALTER TABLE discounts ADD CONSTRAINT chk_discount_percentage CHECK (discount_percentage BETWEEN 0.00 AND 100.00);
>
>
> ```

### Part C: Foreign Key Constraints with Actions

9. "When a department is deleted, all employees in that department should have their `department_id` set to NULL (they become unassigned)."

10. "When a customer is deleted, prevent the deletion if the customer has any orders."

11. "When an author is deleted, all their blog posts should be deleted automatically."

12. "When a course is deleted, all enrollments for that course should be removed."

> [!NOTE]
> ***Your SQL***
>
> ```sql
> ALTER TABLE employees ADD CONSTRAINT fk_employees_department FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE SET NULL;
ALTER TABLE orders ADD CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE RESTRICT;
ALTER TABLE posts ADD CONSTRAINT fk_posts_author FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE CASCADE;
ALTER TABLE enrollments ADD CONSTRAINT fk_enrollments_course FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE;
>
>
> ```

---

## Submission Checklist

- [ ] Exercise 1: `.sql` file with all CREATE TABLE statements + written justifications
- [ ] Exercise 2: All 12 theory review answers
- [ ] Exercise 3: Hotel booking schema with all tables and explanations
- [ ] Exercise 4: Data type selections with justifications for all 15 columns
- [ ] Exercise 5: All 12 constraints written in valid PostgreSQL syntax
