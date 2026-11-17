# Week 46 — Transactions and Concurrency: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Write an Order Transaction

Write a complete transaction that processes a new order in TrailShop:

1. Insert a new order for an existing customer.
2. Insert 2–3 order items.
3. Decrease `stock_quantity` for each ordered product.
4. Include a check that stock doesn't go negative (if it would, ROLLBACK).
5. Use a SAVEPOINT before the stock update step.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > BEGIN;

-- 1. Order for an existing customer
INSERT INTO orders (customer_id, order_date, status)
VALUES (5, NOW(), 'pending')
RETURNING order_id;            -- note the id, e.g. 1001

-- 2. Order items (replace 1001 with the returned id)
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
SELECT 1001, p.product_id, v.qty, p.price
FROM (VALUES (101, 2), (102, 1), (103, 3)) AS v(pid, qty)
JOIN products p ON p.product_id = v.pid;

SAVEPOINT before_stock_update;

-- 3. Decrease stock
UPDATE products p
SET stock_quantity = p.stock_quantity - oi.quantity
FROM order_items oi
WHERE oi.order_id = 1001 AND oi.product_id = p.product_id;

-- 4. Check: any negative stock?
SELECT product_id, stock_quantity FROM products WHERE stock_quantity < 0;
-- If this returns rows:
--   ROLLBACK TO SAVEPOINT before_stock_update;   (undo only the stock update)
--   ROLLBACK;                                    (cancel the whole order)
-- If it returns no rows:
COMMIT;
   >
   >
   > ```

Test the transaction by:

- Running it successfully with sufficient stock.
- Running it where stock is insufficient — verify the ROLLBACK works.

### Task 1.2: Observe READ COMMITTED Behavior (Optional)

> [!TIP]
> **Recommended practice.** Do Tasks 1.2–1.4. They are not required to finish the TrailShop project. They prepare you for the exams.

Open two psql sessions (or two query windows). Demonstrate a non-repeatable read:

1. Session A: `BEGIN;` then `SELECT stock_quantity FROM products WHERE product_id = 101;`
2. Session B: `BEGIN;` then `UPDATE products SET stock_quantity = stock_quantity - 5 WHERE product_id = 101;` then `COMMIT;`
3. Session A: `SELECT stock_quantity FROM products WHERE product_id = 101;` (observe different value)
4. Session A: `COMMIT;`

Document what you observe and explain why it happens at READ COMMITTED.

### Task 1.3: Observe Locking (Optional)

Open two sessions and demonstrate row-level locking:

1. Session A: `BEGIN;` then `SELECT * FROM products WHERE product_id = 101 FOR UPDATE;`
2. Session B: Try `UPDATE products SET price = 99.99 WHERE product_id = 101;` — observe it waits.
3. Session A: `COMMIT;` — observe Session B proceeds.

Document the behavior.

### Task 1.4: Experiment with Isolation Levels (Optional)

1. Repeat Task 1.2, but set Session A to `REPEATABLE READ`. Does the non-repeatable read still occur?

> [!NOTE]
> **_Your Answer_**
>
> _(No — same value both times.)_
>
> 2. In a REPEATABLE READ transaction, try to UPDATE a row that another transaction has already committed a change to. What error do you get?

> [!NOTE]
> **_Your Answer_**
>
> _(ERROR: could not serialize access due to concurrent update.)_
>
> 3. Document the differences between READ COMMITTED and REPEATABLE READ behavior.

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is a transaction, and why is it important for data integrity?

> [!NOTE]
> **_Your Answer_**
>
> _(A transaction = one logical unit of work. Ensures all changes succeed together or not at all → keeps data correct.)_

2. Explain the Atomicity property using a real-world example (not from the lecture notes).

> [!NOTE]
> **_Your Answer_**
>
> _(Atomicity = all-or-nothing. Example: bank transfer — if it fails halfway, neither account changes. Money doesn't vanish.)_
>
> 3. What does Isolation guarantee? Why is it needed in a multi-user system?

> [!NOTE]
> **_Your Answer_**
>
> _(Isolation = transactions don't interfere. Needed so multiple users don't overwrite each other or see half-done changes.)_

4. What is the difference between COMMIT and ROLLBACK?

> [!NOTE]
> **_Your Answer_**
>
> _(COMMIT = save changes permanently. ROLLBACK = discard all changes since transaction started.)_

5. Explain what a SAVEPOINT is and give a scenario where it's useful.

> [!NOTE]
> **_Your Answer_**
>
> _(SAVEPOINT = bookmark inside a transaction. Roll back only part, keep the rest — good when one step fails but you don't want to lose everything.)_
>
> 6. What is a dirty read? Why does PostgreSQL never allow it?

> [!NOTE]
> **_Your Answer_**
>
> _(Dirty read = seeing uncommitted changes from another transaction. If that other rolls back, you read data that never existed. PostgreSQL never allows this.)_

7. Describe the lost update problem. How can SELECT FOR UPDATE prevent it?

> [!NOTE]
> **_Your Answer_**
>
> _(Lost update = two people edit same row, second overwrites first. `SELECT FOR UPDATE` locks the row so second must wait → can't overwrite.)_

8. What is the default isolation level in PostgreSQL? What concurrency problems does it still allow?

> [!NOTE]
> **_Your Answer_**
>
> _(Default = READ COMMITTED. Still allows: non-repeatable reads, phantom reads.)_

9. Explain MVCC in one paragraph: what is it, and what advantage does it provide?

> [!NOTE]
> **_Your Answer_**
>
> _(MVCC keeps multiple row versions. Readers see a consistent snapshot; writers create new versions. Readers don't block writers, writers don't block readers → better concurrency.)_

10. What is a deadlock, and how does PostgreSQL handle it when one is detected?

> [!NOTE]
> **_Your Answer_**
>
> _(Deadlock = two transactions wait for each other's locks forever. PostgreSQL detects it and **aborts one** to break the cycle.)_

---

## 3. Scenario Analysis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each scenario, identify: (a) which concurrency problem occurs, and (b) at which isolation level(s) it can happen.

### Scenario A (a)Non-repeatable read (b)READ COMMITTED

Transaction 1 reads customer 5's order count (3 orders). Transaction 2 inserts a new order for customer 5 and commits. Transaction 1 reads customer 5's order count again and gets 4.

### Scenario B (a) Dirty read (b) Not possible in PostgreSQL

Transaction 1 reads product 101's price (€89.99). Transaction 2 updates product 101's price to €79.99 but then rolls back. If Transaction 1 had somehow seen €79.99, what problem would that be?

### Scenario C (a) Lost update (b) READ COMMITTED

Two warehouse workers simultaneously check stock for product 101 (both see 10). Worker A ships 3 units and sets stock to 7. Worker B ships 4 units and sets stock to 6. The correct stock should be 3.

---

## 4. Transaction Writing (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write complete SQL transactions for the following TrailShop scenarios. Include BEGIN, appropriate operations, error handling (SAVEPOINT where useful), and COMMIT.

### Transaction 1: Bulk Price Update

Update the price of all products in the "Footwear" category by +10%. If any product's new price would exceed €200, rollback the entire update.

### Transaction 2: Customer Account Deletion

Delete a customer and all their associated data:

1. Delete their order_items (for all their orders)

> [!NOTE]
> **_Your Answer_**
>
> BEGIN;
UPDATE products SET price = price * 1.10
WHERE category_id = (SELECT category_id FROM categories WHERE name = 'Footwear');
SELECT COUNT(*) FROM products
WHERE category_id = (SELECT category_id FROM categories WHERE name = 'Footwear')
  AND price > 200;
-- count > 0 → ROLLBACK; otherwise → COMMIT;
>
> 2. Delete their orders

> [!NOTE]
> **_Your Answer_**
>
> BEGIN;
SAVEPOINT s1;
DELETE FROM order_items WHERE order_id IN (SELECT order_id FROM orders WHERE customer_id = 5);
SAVEPOINT s2;
DELETE FROM orders WHERE customer_id = 5;
SAVEPOINT s3;
DELETE FROM customers WHERE customer_id = 5;
-- on an error at any step: ROLLBACK TO <last savepoint> (or ROLLBACK)
COMMIT;
>
> 3. Delete the customer record

> [!NOTE]
> **_Your Answer_**
>
> BEGIN;
SELECT product_id, stock_quantity FROM products
WHERE product_id IN (101,102,103) FOR UPDATE;

UPDATE products SET stock_quantity = stock_quantity + 50 WHERE product_id = 101;
UPDATE products SET stock_quantity = stock_quantity + 30 WHERE product_id = 102;
UPDATE products SET stock_quantity = stock_quantity + 20 WHERE product_id = 103;

SELECT product_id FROM products
WHERE product_id IN (101,102,103) AND stock_quantity > 200;
-- rows returned → ROLLBACK; otherwise → COMMIT;

Use SAVEPOINTs between each step in case of errors.

### Transaction 3: Inventory Restock

Restock 3 products (add specific quantities to stock). Use SELECT FOR UPDATE to lock the rows first, preventing lost updates. Include a check that the new stock doesn't exceed a maximum (e.g., 200).

---

## 5. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. In PostgreSQL, a single SQL statement outside of an explicit BEGIN/COMMIT is not a transaction.
   FALSE — Every single statement runs in its own auto-transaction
2. ROLLBACK undoes all changes made since the last SAVEPOINT.
   FALSE — ROLLBACK alone undoes everything; ROLLBACK TO sp undoes only after that savepoint
3. At READ COMMITTED isolation level, a transaction can never see another transaction's uncommitted changes.
   TRUE — READ COMMITTED never shows uncommitted data
4. REPEATABLE READ in PostgreSQL prevents phantom reads.
   TRUE — PostgreSQL's REPEATABLE READ prevents phantom reads too
5. SELECT FOR UPDATE blocks other SELECT statements on the same rows.
   FALSE — Regular SELECT still works; only writes/SELECT FOR UPDATE wait
6. If a deadlock occurs, PostgreSQL will wait indefinitely for it to resolve.
   FALSE — PostgreSQL detects and breaks deadlocks immediately
7. MVCC means PostgreSQL keeps multiple versions of each row, allowing readers and writers to work concurrently without blocking.
   TRUE — That's exactly what MVCC does
8. Once a transaction is committed, its changes can be undone with ROLLBACK.
   FALSE — Committed changes are permanent; can't roll back

---

## Submission Checklist

**Required**

- [ ] Order transaction written and tested, including the insufficient-stock rollback (Task 1.1)
- [ ] Theory review questions answered
- [ ] True/False questions answered with a short explanation

**Recommended practice**

- [ ] READ COMMITTED, locking, and isolation-level experiments documented (Tasks 1.2–1.4)
- [ ] Scenario analysis completed
- [ ] Extra transactions written
