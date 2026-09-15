Week 37 — Exercises & Project Task
Important

How to Complete These Exercises Write your answers directly in the highlighted Your Answer / Your SQL fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 37 Theory material. Complete all sections.

Part 1: TrailShop Project Task
Task 1: Identify Keys
Using the products, categories, and customers tables shown in Section 2 of this week's Theory material, answer:

What is the primary key of the products table? Why is it a good choice?
Note

Your Answer

product_id is the primary key. It is a good choice because it is unique for every product, never changes, has no business meaning  is simple to use, and guarantees each row can be identified reliably .)

What is the primary key of the categories table?
Note

Your Answer

(category-id.)

What is the foreign key in the products table? What does it reference?
Note

Your Answer

(category -id is the foregin key .It references category-id in the categories table.)

Is name in products a candidate key? Under what assumption? What would make it unsuitable as a primary key?
Note

Your Answer

(Yes, name could be a candidate key if all product names are guaranteed to be unique. It is unsuitable as a primary key because names can change, they are longer/slower than integers, and uniqueness is not always guaranteed in practice.)

Give an example of a superkey for the products table that is NOT a candidate key. Explain why it's not minimal.
Note

Your Answer

({product_id, name} is a superkey but not a candidate key. It is not minimal because product_id alone is already sufficient to uniquely identify a row — adding name is redundant.)

Give an example of a composite key using a hypothetical order_items table. Explain why neither column alone would be sufficient.
Note

Your Answer

(order_id, product_id) together form a composite primary key. Neither column alone is sufficient because one order can contain multiple products, and one product can appear in multiple orders — only the combination is unique.)

Is email in customers a candidate key? What makes it different from customer_id as a PK choice? (See Section 6.9 on natural vs surrogate keys.)
Note

Your Answer

(Yes, email is a candidate key .Unlike customer_id (a surrogate key, emails can change, they are longer, and they have real-world meaning — making them less ideal as a primary key.)

Task 2: Define Business Rules
List 5 business rules for TrailShop. For each rule, specify:

The rule in plain English
Which constraint type(s) would enforce it
Which table and column the constraint applies to
The SQL syntax for the constraint
Example:

Business Rule	Constraint Type	Table.Column	SQL
Every product must have a price greater than zero	CHECK	products.price	CHECK (price > 0)
...	...	...	...
Think about rules for customers, orders, and categories — not just products.

Note

Your Answer

(List your 5 business rules with constraint types, table/column, and SQL syntax.
Business Rule	                          Constraint Type              	Table.Column	                               SQL	
Every category must have a unique name  	UNIQUE	               categories.category_name	                UNIQUE (category_name)	
Customer email addresses must be unique 	UNIQUE	                   customers.email                      UNIQUE (email)	
Stock quantity cannot be negative	        CHECK                   	products.stock_quantity             	CHECK (stock_quantity >= 0)	
Every product must have a name	          NOT NULL	                 products.name	                      NOT NULL	
Order quantity must be greater than zero 	CHECK	                     order_items.quantity               	CHECK (quantity > 0))

Task 3: Integrity Violations
For each SQL statement below, predict whether it will succeed or fail. If it fails, explain which integrity rule or constraint is violated and what error message you'd expect. Assume the schema from Section 9.8 of the Theory material.

-- Statement A
INSERT INTO categories (category_id, category_name)
VALUES (NULL, 'Cycling');

-- Statement B
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'AeroLite Tent', 279.00, 10, 2);

-- Statement C
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (110, 'BudgetBoots', -5.00, 25, 1);

-- Statement D
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (103, 'Duplicate Shoes', 99.99, 5, 3);

-- Statement E
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (111, 'CloudWalker Sandals', 65.00, 40, 10);

-- Statement F
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (112, NULL, 89.99, 20, 1);

-- Statement G
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (113, 'LightStep Shoes', 149.00, -3, 1);

-- Statement H
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1001, 101, 0, 189.50);
Note

Your Answer

(For each statement A–H, write SUCCESS or FAIL and explain any violation.)
Statement  	Result	   Explanation	
A	         FAIL	       Violates Entity Integrity — category_id (PK) cannot be NULL.	
B	       SUCCESS	     Valid — category_id 2 exists, all constraints satisfied.	
C          FAIL	       Violates CHECK constraint — price cannot be negative (-5.00).	
D	         FAIL	       Violates PK Uniqueness — product_id 103 already exists.	
E	         FAIL      	 Violates Referential Integrity — category_id 10 does not exist in categories.	
F          FAIL	       Violates NOT NULL constraint — product name cannot be NULL.	
G	         FAIL	       Violates CHECK constraint — stock_quantity cannot be negative (-3).	
H	         FAIL	       Violates CHECK constraint — quantity must be > 0 (0 is invalid).


Task 4: Foreign Key Actions
Consider the following scenario using the schema from Theory Section 9.8:

You want to delete category 2 ("Camping") from the categories table. Products 102 and 106 reference this category. What happens with:

ON DELETE RESTRICT?
ON DELETE CASCADE?
ON DELETE SET NULL? (Assume category_id in products allows NULL for this question)
Which foreign key action would you recommend for the TrailShop products.category_id → categories.category_id relationship? Justify your choice in 2–3 sentences.

Note

Your Answer

(ON DELETE RESTRICT: The deletion fails — PostgreSQL blocks it because products 102 and 106 still reference category 2. You must reassign or delete those products first.

ON DELETE CASCADE: Category 2 is deleted, and products 102 and 106 are also automatically deleted.

ON DELETE SET NULL: Category 2 is deleted; products 102 and 106 remain but their category_id is set to NUL.)

Part 2: Theory Review Questions
Answer each question in 2–4 sentences unless otherwise specified. Reference the Theory material sections as needed.

Short-Answer Questions
Q1. Define the following terms in your own words: relation, tuple, attribute, domain. Give one TrailShop example for each.

Note

Your Answer

(Relation = A table that stores a collection of related data. Example: the products table.

Tuple = One complete row or record in a relation. Example: one specific product such as "AeroLite Tent".

Attribute = A column or property that describes the relation. Example: the price of a product.

Domain = The set of all allowed values for an attribute. Example: price must be a positive number.)

(See Sections 2 and 3 of this week's Theory material.)

Q2. What makes a candidate key different from a primary key? Can a table have more than one candidate key?

Note

Your Answer

(A candidate key can uniquely identify rows; a primary key is the one chosen as the main ID. Yes, a table can have multiple candidate keys — only one is the PK.)

(See Section 6 of this week's Theory material.)

Q3. Explain entity integrity in your own words. Why can't a primary key be NULL?

Note

Your Answer

(Entity integrity means every row must have a unique, non-null PK. PK can't be NULL because NULL means "unknown" — you can't identify a row without a clear value.)

(See Section 8.1 of this week's Theory material.)

Q4. What happens when referential integrity is violated? Give a concrete TrailShop example — show the SQL statement and the expected error.

Note

Your Answer

(Violation = FK references a PK that doesn't exist.)

(See Section 8.2 of this week's Theory material.)

Q5. Explain the difference between a surrogate key and a natural key. Give an example of each for a books table in a library database.

Note

Your Answer

(Surrogate key: Artifical ID,no real-world meaning.Example:book -id 
Natural key: From real data,has meaning.Example :ISBN)

(See Section 6.8–6.9 of this week's Theory material.)

Q6. What is a NULL value? Why is WHERE price = NULL wrong? What should you write instead?

Note

Your Answer

(NULL = unknown/missing value.
price = NULL is wrong because NULL ≠ anything, even itself.
Use: price IS NULL or price IS NOT NULL)

(See Section 7 of this week's Theory material.)

Q7. What is a junction table? When is it needed? Give an example.

Note

Your Answer

(A junction table connects two tables in a many-to-many relationship.
Needed when both sides can have multiple matches.
Example: order_items links orders and products)

(See Section 12.3 of this week's Theory material.)

Q8. Describe the three types of relationships (1:1, 1:N, M:N). For each, give one TrailShop example.

Note

Your Answer

(1:1 — One ↔️ One. Example: product ↔️ product_details

* 1:N — One ↔️ Many. Example: category ↔️ products

* M:N — Many ↔️ Many. Example: orders ↔️ products )

(See Section 12 of this week's Theory material.)

Q9. What is the difference between ON DELETE CASCADE and ON DELETE RESTRICT? When would you use each?

Note

Your Answer

(ON DELETE CASCADE: Deletes child rows automatically when the parent is deleted. Use when child records cannot exist without the parent.

ON DELETE RESTRICT: Blocks deletion if child rows still exist. Use when both tables are independent and you want to prevent accidental data loss.)

(See Section 10 of this week's Theory material.)

Q10. Explain what "atomic entries" means in the context of relation properties. Give an example of a violation.

Note

Your Answer
(Atomic entries means each cell contains exactly one value — never multiple values.
Violation example: Storing "Boots, Jackets" in one category column.)

(See Section 5.3 of this week's Theory material.)

True/False
For each statement, write True or False and correct any false statements.

False-A superkey is always a candidate key.
True -A primary key can consist of more than one column.
False-NULL = NULL evaluates to TRUE in SQL.
False-A foreign key must always be NOT NULL.
True- Referential integrity ensures that every FK value matches an existing PK value (or is NULL).D
False-The degree of a relation is the number of rows.
Matching Exercise
Match each term (1–12) with its definition (A–L).

#	Term
1	Superkey
2	Candidate key
3	Composite key
4	Foreign key
5	Alternate key
6	Surrogate key
7	Natural key
8	Orphan record
9	Domain
10	Junction table
11	Cardinality
12	COALESCE
Letter	Definition
A	The set of all permitted values for an attribute
B	A key composed of two or more attributes
C	A row whose FK references a non-existent PK — forbidden by referential integrity
D	An artificial key with no business meaning (e.g., auto-generated ID)
E	A candidate key not chosen as the primary key
F	Any set of attributes that uniquely identifies every tuple
G	A minimal superkey — no attribute can be removed without losing uniqueness
H	A column that references the primary key of another table
I	The number of tuples (rows) in a relation
J	A key drawn from real-world data with business meaning
K	A table implementing a many-to-many relationship
L	A SQL function that returns the first non-NULL argument
Note

Your Answers

#	Your Match
1	F
2	G
3	B
4	H
5	E
6	D
7	J
8	C
9	A
10	K
11	I
12	L
Part 3: SQL Practice — Constraints in Action
These exercises test your understanding of constraints. You do NOT need to run these in PostgreSQL (but you may if you'd like to verify your answers).

Exercise 3.1: Predict the Outcome
Given the following table definitions:

CREATE TABLE departments (
    dept_id   INTEGER      PRIMARY KEY,
    dept_name VARCHAR(50)  NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id    INTEGER       PRIMARY KEY,
    name      VARCHAR(100)  NOT NULL,
    salary    NUMERIC(10,2) NOT NULL CHECK (salary >= 0),
    dept_id   INTEGER       NOT NULL REFERENCES departments(dept_id)
);
Assume these rows already exist:

INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO departments VALUES (2, 'Marketing');
INSERT INTO employees VALUES (100, 'Alice', 75000, 1);
INSERT INTO employees VALUES (101, 'Bob', 65000, 2);
For each statement below, predict: SUCCESS or FAIL? If fail, name the violated constraint.

-- 1
INSERT INTO employees VALUES (102, 'Carol', 70000, 1);
SUCCESS  No violations-all values valid
-- 2
INSERT INTO employees VALUES (103, 'Dan', -5000, 1);
FAIL CHECK( salary =0) -negative salary not allowed
-- 3
INSERT INTO employees VALUES (100, 'Eve', 80000, 2);
FAIL PRIMARY KEY -emp-id 100 already exists
-- 4
INSERT INTO employees VALUES (104, 'Frank', 60000, 5);
FAIL FOREIGN KEY -dept -id 5 doesn't exist in departments
-- 5
INSERT INTO departments VALUES (3, 'Engineering');
FAIL UNIQUE- 'Engineering already exists
-- 6
INSERT INTO employees VALUES (105, NULL, 55000, 2);
FAIL NOT NULL-name cannot be NULL
-- 7
DELETE FROM departments WHERE dept_id = 1;
FAIL FOREIGN KEY RESTRICT -employees 100,102 reference dept-id 1
-- 8
INSERT INTO employees VALUES (106, 'Grace', 0, 2);
SUCCESS CHECK allows salary = 0 (>= 0)
Exercise 3.2: Write the Constraints
Given these business rules for a bookstore database, write the CREATE TABLE statements with appropriate constraints:

Every book has a unique ISBN (13 characters), a title (required), a price (must be positive), and a publication year.
Every author has an ID, a first name (required), and a last name (required).
A book can have multiple authors, and an author can write multiple books.
Every book belongs to exactly one genre. Genres have an ID and a unique name.
Publication year must be between 1450 and the current year.
(Hint: you'll need at least 4 tables, including a junction table for the M:N relationship.)
-- Genres table
CREATE TABLE genres (
    genre_id INTEGER PRIMARY KEY,
    genre_name VARCHAR(50) NOT NULL UNIQUE
);

-- Authors table
CREATE TABLE authors (
    author_id INTEGER PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL
);

-- Books table
CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    publication_year INTEGER CHECK (publication_year BETWEEN 1450 AND EXTRACT(YEAR FROM CURRENT_DATE)),
    genre_id INTEGER NOT NULL REFERENCES genres(genre_id)
);

-- Junction table: Book-Author (M:N relationship)
CREATE TABLE book_authors (
    isbn CHAR(13) REFERENCES books(isbn),
    author_id INTEGER REFERENCES authors(author_id),
    PRIMARY KEY (isbn, author_id)
);

Part 4: Design Exercise — Library System
A small public library needs a database. Here is a description of their requirements:

The library has a collection of books. Each book has an ISBN, a title, a publication year, and belongs to one genre (Fiction, Non-Fiction, Science, History, etc.). The library may own multiple copies of the same book — each copy has a unique barcode sticker.

The library has registered members. Each member has a member number, name, email, and phone. Members can borrow copies. Each borrowing records which member borrowed which copy, the borrow date, the due date, and the return date (NULL if not yet returned).

Rules:

A member can borrow at most 5 copies at any given time.
The due date is always 14 days after the borrow date.
A copy cannot be borrowed if it's currently not returned (return_date IS NULL).
Your Tasks
Identify the tables you would need 

(genres: genre_id, genre_name

* books: isbn, title, publication_year, genre_id

* copies: copy_id, isbn, status

* members: member_id, name, email, phone

* borrowings: borrowing_id, copy_id, member_id, borrow_date, due_date, return_date ).

Identify the primary key for each table. Are they surrogate or natural keys? Justify your choices.
 Primary Keys & Type
* genres.genre_id → Surrogate key (auto-generated ID)

* books.isbn → Natural key (real-world meaning)

* copies.copy_id → Surrogate key

* members.member_id → Surrogate key

* borrowings.borrowing_id → Surrogate key
  
Identify all foreign keys and the tables they reference.
 Foreign Keys
* books.genre_id → genres.genre_id

* copies.isbn → books.isbn

* borrowings.copy_id → copies.copy_id

* borrowings.member_id → members.member_id

Identify any candidate keys beyond the primary key (alternate keys).
Candidate Keys
* books: isbn (PK)

* members: member_id (PK), email (alternate key — unique)

* genres: genre_id (PK), genre_name (alternate key — unique)

List the business rules from the description and map each to a constraint type. Which rules cannot be enforced by simple constraints?
Note

Your Answer

(Business Rules & Constraints
Your Answer:
* ISBN must be unique → PRIMARY KEY 

* Title & member name required → NOT NULL 

* Publication year 1450–current → CHECK 

* Member max 5 copies at once → Cannot enforce with simple constraints 

* Due date = borrow_date + 14 days → Default value 

* Cannot borrow if not returned → Cannot enforce with simple constraints 
⑥ CREATE TABLE Statements
CREATE TABLE books (
    isbn CHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    publication_year INTEGER CHECK (publication_year BETWEEN 1450 AND EXTRACT(YEAR FROM CURRENT_DATE)),
    genre_id INTEGER NOT NULL REFERENCES genres(genre_id)
);

CREATE TABLE copies (
    copy_id INTEGER PRIMARY KEY,
    isbn CHAR(13) NOT NULL REFERENCES books(isbn) ON DELETE CASCADE,
    status VARCHAR(20) DEFAULT 'Available'
);

CREATE TABLE borrowings (
    borrowing_id INTEGER PRIMARY KEY,
    copy_id INTEGER NOT NULL REFERENCES copies(copy_id),
    member_id INTEGER NOT NULL REFERENCES members(member_id),
    borrow_date DATE NOT NULL DEFAULT CURRENT_DATE,
    due_date DATE NOT NULL DEFAULT (CURRENT_DATE + INTERVAL '14 days'),
    return_date DATE
);
Write the CREATE TABLE statements for at least the books, copies, and borrowings tables with full constraints.
Submission Checklist
 Task 1: Key identification answers (Part 1)
 Task 2: Business rules table with 5 rules (Part 1)
 Task 3: Integrity violation predictions with explanations (Part 1)
 Task 4: Foreign key action analysis (Part 1)
 Theory Review Questions answered (Part 2)
 SQL Practice — constraint predictions and bookstore CREATE TABLE (Part 3)
 Library System design exercise (Part 4)
