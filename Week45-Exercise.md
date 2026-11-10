# Week 45 — Normalization: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Analyze a Denormalized Table

Consider this denormalized table that TrailShop's old system used:

```
supplier_products(
    supplier_id, supplier_name, supplier_email, supplier_city,
    product_id, product_name, category_id, category_name,
    supply_price, lead_time_days
)
```

Sample data:

| supplier_id | supplier_name | supplier_email      | supplier_city | product_id | product_name    | category_id | category_name | supply_price | lead_time_days |
| ----------- | ------------- | ------------------- | ------------- | ---------- | --------------- | ----------- | ------------- | ------------ | -------------- |
| 1           | NordGear Oy   | info@nordgear.fi    | Helsinki      | 101        | Trail Runner X  | 10          | Footwear      | 45.00        | 7              |
| 1           | NordGear Oy   | info@nordgear.fi    | Helsinki      | 102        | Summit Boot     | 10          | Footwear      | 62.00        | 7              |
| 2           | OutdoorPro AB | sales@outdoorpro.se | Stockholm     | 201        | Rain Jacket Pro | 30          | Clothing      | 55.00        | 14             |
| 2           | OutdoorPro AB | sales@outdoorpro.se | Stockholm     | 101        | Trail Runner X  | 10          | Footwear      | 43.00        | 10             |

**Tasks:**

1. Identify the primary key of this table.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Primary Key: (supplier_id, product_id)
        ALTER TABLE supplier_products ADD PRIMARY KEY (supplier_id, product_id);
   >
   >
   > ```
2. Identify any insertion, update, and deletion anomalies (give specific examples).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Insertion Anomaly: Cannot add a new supplier until they have a product
     -- Update Anomaly: Changing supplier_city requires updating every row for that supplier
     -- Deletion Anomaly: Deleting all products from a supplier deletes the supplier too
   >
   >
   > ```
3. Normalize this table to 3NF. Show each step (1NF, then 2NF, then 3NF) with the resulting tables.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   >-- Step 1NF: Already satisfied — one value per cell
    -- Step 2NF: Remove partial dependencies
   CREATE TABLE Suppliers (
    supplier_id INT PRIMARY KEY,
    supplier_name VARCHAR(100) NOT NULL,
    supplier_email VARCHAR(100),
    supplier_city VARCHAR(50)
);

CREATE TABLE Categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
);

CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category_id INT REFERENCES Categories(category_id)
);

CREATE TABLE Supplier_Products (
    supplier_id INT REFERENCES Suppliers(supplier_id),
    product_id INT REFERENCES Products(product_id),
    supply_price DECIMAL(10,2) NOT NULL,
    lead_time_days INT,
    PRIMARY KEY (supplier_id, product_id)
);
-- Step 3NF: No transitive dependencies — all tables are in 3NF
   >
   >
   > ```

### Task 1.2: Review Your Existing TrailShop Schema

1. For each table, confirm it is in 3NF by verifying:
   - No partial dependencies (check tables with composite keys or surrogate keys)
   - No transitive dependencies
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- 3NF Verification:
     -- 1. No partial dependencies.
     -- All non-key attributes depend on the entire primary key.
     -- 2. No transitive dependencies.
     -- Non-key attributes do not depend on other non-key attributes.
     -- 3. All tables have atomic values (1NF).
     -- Therefore, all tables satisfy 3NF.
   >
   >
   > ```
2. Explain why `unit_price` in `order_items` is NOT a normalization violation.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >  -- unit_price stores the price AT THE TIME OF ORDER
      -- Product prices change over time — we need the historical value
      -- This is intentional, not redundant
   >
   > ```

### Task 1.3: Add a Suppliers Table (Optional Extension)

You may skip this task. Suppliers are not part of the TrailShop project schema.

If TrailShop wanted to track suppliers:

1. Design a normalized schema that includes suppliers. Consider:
   - Each product can have multiple suppliers
   - Each supplier offers different prices and lead times
   - Each supplier has contact info (name, email, city)
2. Write the `CREATE TABLE` statements.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > CREATE TABLE Suppliers (
    supplier_id INT PRIMARY KEY,
    supplier_name VARCHAR(100) NOT NULL,
    supplier_email VARCHAR(100),
    supplier_city VARCHAR(50)
);

CREATE TABLE Categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
);

CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category_id INT REFERENCES Categories(category_id)
);

CREATE TABLE Supplier_Products (
    supplier_id INT REFERENCES Suppliers(supplier_id),
    product_id INT REFERENCES Products(product_id),
    supply_price DECIMAL(10,2) NOT NULL,
    lead_time_days INT,
    PRIMARY KEY (supplier_id, product_id)
);
-- ✅ Verified in 3NF
   >
   
   >
   > ```
3. Verify your design is in 3NF.

> [!NOTE]
> **_Your Answer_**
>
> _(The design satisfies 3NF because:
1. Each table has atomic values (1NF)
2. All non-key columns depend fully on the primary key only (2NF)
3. No transitive dependencies exist — all non-key columns depend only on the primary key (3NF))_

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is normalization, and what problem does it solve?

> [!NOTE]
> **_Your Answer_**
>
> _(Organizing data into tables to reduce redundancy and prevent update/insert/delete anomalies. It keeps data consistent and makes changes easier.)_

2. Explain the difference between an insertion anomaly and a deletion anomaly, using an example.

> [!NOTE]
> **_Your Answer_**
>
> _(Insert: Can't add data without also adding unrelated data (e.g., can't add a supplier with no products). **Delete:** Deleting one thing accidentally deletes something else (e.g., deleting the last product deletes the supplier.)_

3. What is the difference between a partial dependency and a transitive dependency?

> [!NOTE]
> **_Your Answer_**
>
> _(Partial: Column depends on only part of a composite PK. Transitive: Non-key column depends on another non-key column, not directly on the PK.)_

4. State the "informal rule" for 3NF. Explain what each part means.

> [!NOTE]
> **_Your Answer_**
>
> _(All attributes must depend on the key, the whole key, and nothing but the key."** — No partial dependencies, no transitive dependencies.)_

5. Why can partial dependencies only occur in tables with composite primary keys?

> [!NOTE]
> **_Your Answer_**
>
> _(A single-column primary key has no "part" — everything depends on that one key. Only composite keys can have columns that depend on just one part.)_

6. What is BCNF, and how does it differ from 3NF? In what rare case can a table be in 3NF but not BCNF?

> [!NOTE]
> **_Your Answer_**
>
> _(BCNF is stricter: every determinant must be a candidate key. 3NF allows some exceptions where a non-key determines another non-key. Rare case: multiple overlapping candidate keys → 3NF but not BCNF.)_

7. Give two legitimate reasons to denormalize a database.

> [!NOTE]
> **_Your Answer_**
>
> _(Improve performance  — fewer JOINs for frequent queries. 2. Simplify reporting  — pre-computed values for read-heavy systems.)_

---

## 3. Normalization Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Normalize each table to 3NF. Show your work for each step.

### Table 1: event_registrations

| event_id | event_name     | event_date | venue_id | venue_name      | venue_city | attendee_id | attendee_name | ticket_type | ticket_price |
| -------- | -------------- | ---------- | -------- | --------------- | ---------- | ----------- | ------------- | ----------- | ------------ |
| E1       | Trail Run 2025 | 2025-06-15 | V1       | City Park Arena | Helsinki   | A1          | Matti Aho     | VIP         | 75.00        |
| E1       | Trail Run 2025 | 2025-06-15 | V1       | City Park Arena | Helsinki   | A2          | Sara Lund     | Standard    | 35.00        |
| E2       | Mountain Fest  | 2025-07-20 | V2       | Summit Center   | Tampere    | A1          | Matti Aho     | Standard    | 40.00        |

**Business rules:**

- Each event has one name, date, and venue.
- Each venue has one name and city.
- Each attendee has one name.
- Ticket price depends on the event and ticket type combination.

**Steps:**

1. Identify the PK
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- PK: (attendee_id, event_id)
     CREATE TABLE Event_Registrations (
    attendee_id CHAR(2) REFERENCES Attendees(attendee_id),
    event_id CHAR(2) REFERENCES Events(event_id),
    ticket_type VARCHAR(20),
    PRIMARY KEY (attendee_id, event_id),
    FOREIGN KEY (event_id, ticket_type) 
        REFERENCES Ticket_Prices(event_id, ticket_type)
);
   >
   > ```
2. Check 1NF, 2NF, 3NF
3. Decompose as needed
4. Write final CREATE TABLE statements
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- PK: (store_id, product_id)
     CREATE TABLE Stores (store_id CHAR(2) PRIMARY KEY, store_name VARCHAR(100), store_city VARCHAR(50));
     CREATE TABLE Suppliers (supplier_name VARCHAR(100) PRIMARY KEY);
     CREATE TABLE Products (product_id INT PRIMARY KEY, product_name VARCHAR(100), category VARCHAR(50), supplier_name VARCHAR(100) REFERENCES Suppliers(supplier_name));
     CREATE TABLE Store_Inventory (store_id CHAR(2), product_id INT, quantity INT, reorder_level INT, PRIMARY KEY (store_id, product_id));
   >
   >
   > ```

### Table 2: store_inventory

| store_id | store_name         | store_city | product_id | product_name    | category | quantity | reorder_level | supplier_name |
| -------- | ------------------ | ---------- | ---------- | --------------- | -------- | -------- | ------------- | ------------- |
| S1       | TrailShop Helsinki | Helsinki   | 101        | Trail Runner X  | Footwear | 50       | 10            | NordGear Oy   |
| S1       | TrailShop Helsinki | Helsinki   | 201        | Rain Jacket Pro | Clothing | 20       | 5             | OutdoorPro AB |
| S2       | TrailShop Tampere  | Tampere    | 101        | Trail Runner X  | Footwear | 30       | 8             | NordGear Oy   |

**Business rules:**

- Each store has one name and city.
- Each product has one name, category, and supplier.
- Quantity and reorder_level depend on the specific store-product combination.

**Steps:** Same as above.

---

## 4. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. A table with a single-column primary key can violate 2NF.
   FALSE — With a single-column PK, all columns depend on the whole key. Partial dependencies can only exist with composite keys.
2. If a table is in 3NF, it is automatically in 2NF.
   TRUE — 3NF requires 2NF first. Every table in 3NF satisfies all 2NF rules.
3. Normalization always improves query performance.
   FALSE — More tables mean more JOINs. Sometimes normalization slows down read queries.
4. Storing a customer's current address in the orders table is always a normalization violation.
   FALSE — Orders need the *historical* address at purchase time. This is intentional, not redundant.
5. BCNF is stricter than 3NF — every table in BCNF is also in 3NF.
   TRUE — BCNF adds stricter rules on top of 3NF. If you meet BCNF, you automatically meet 3NF.
6. Denormalization should be done before normalization to save time.
   FALSE — Always normalize first, then denormalize only where needed for performance.

---

## Submission Checklist

**Required**

- [ ] Existing TrailShop schema reviewed and confirmed in 3NF (Task 1.2)
- [ ] Theory review questions answered
- [ ] True/False questions answered with a short explanation

**Recommended practice**

- [ ] Denormalized `supplier_products` table analyzed and normalized (Task 1.1)
- [ ] Normalization exercise completed for both tables

**Optional extension**

- [ ] Suppliers schema designed (Task 1.3) — you may skip this
