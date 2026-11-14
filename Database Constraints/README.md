# Primary Keys, Foreign Keys, and Constraints

## 1. What Is a Constraint?

A **constraint** is a **rule** that we put on a table.
The database checks the rule every time we add, change, or delete data.

If the data breaks the rule, the database **refuses** the change and shows an error.

> **Simple idea:** Constraints are like a security guard for your data. Only good data can enter.

**Why do we need constraints?**

| Without constraints | With constraints |
|---|---|
| Two students with the same ID | Every ID is unique |
| A student with no name | Name is required |
| An order for a customer that does not exist | Order must belong to a real customer |
| Age = -5 | Age must be a valid number |

---

## 2. Primary Key (PK)

### Definition

A **Primary Key** is a column (or group of columns) that **identifies each row in a table in a unique way**.

Think of it like a **national ID number**. No two people have the same one.

### The Rules of a Primary Key

1. **Unique**: no two rows can have the same value.
2. **Not NULL**: it can never be empty.
3. **One per table**: a table can have only **one** primary key.
4. **Stable**: the value should not change over time.

### Example

```sql
CREATE TABLE students (
    student_id  INT PRIMARY KEY,
    name        VARCHAR(50),
    email       VARCHAR(100)
);
```

Data in the table:

| student_id | name  | email          |
|-----------:|-------|----------------|
| 1          | Ahmed | ahmed@mail.com |
| 2          | Sara  | sara@mail.com  |
| 3          | Omar  | omar@mail.com  |

### What happens if we break the rules?

```sql
INSERT INTO students VALUES (1, 'Mona', 'mona@mail.com');
-- ERROR: student_id 1 already exists (duplicate)

INSERT INTO students VALUES (NULL, 'Mona', 'mona@mail.com');
-- ERROR: primary key cannot be NULL
```

### Another Way to Write It

```sql
CREATE TABLE students (
    student_id  INT,
    name        VARCHAR(50),
    email       VARCHAR(100),
    CONSTRAINT pk_students PRIMARY KEY (student_id)
);
```

Here we gave the constraint a **name** (`pk_students`). Named constraints are easier to find and delete later.

### Composite Primary Key

Sometimes **one column is not enough**. We can use **two or more columns together**.

Example: a student can take many courses, and a course has many students.
One student + one course should appear **only once**.

```sql
CREATE TABLE enrollments (
    student_id  INT,
    course_id   INT,
    enroll_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

| student_id | course_id | enroll_date |
|-----------:|----------:|-------------|
| 1          | 101       | 2026-09-01  |
| 1          | 102       | 2026-09-01  |
| 2          | 101       | 2026-09-02  |

- `(1, 101)` and `(1, 102)` are different, so they are OK.
- Adding `(1, 101)` again is **not allowed**.

### How to Choose a Good Primary Key

- Use a **simple number** (like `student_id`).
- Do **not** use names (two people can have the same name).
- Do **not** use values that can change (like phone number or email).

---

## 3. Foreign Key (FK)

### Definition

A **Foreign Key** is a column in one table that **points to the Primary Key of another table**.

It is the way we **connect tables** together.

> **Simple idea:** A foreign key is like a **reference**. "This order belongs to customer number 5."

### Parent Table and Child Table

- **Parent table**: the table that has the Primary Key (the one being pointed to).
- **Child table**: the table that has the Foreign Key (the one that points).

```
customers (PARENT)               orders (CHILD)
+-------------+--------+         +----------+-------------+--------+
| customer_id | name   |         | order_id | customer_id | total  |
+-------------+--------+         +----------+-------------+--------+
| 1  (PK)     | Ahmed  | <-----  | 100      | 1  (FK)     | 250    |
| 2  (PK)     | Sara   | <-----  | 101      | 2  (FK)     | 90     |
| 3  (PK)     | Omar   | <-----  | 102      | 1  (FK)     | 40     |
+-------------+--------+         +----------+-------------+--------+
```

### Example

```sql
CREATE TABLE customers (
    customer_id  INT PRIMARY KEY,
    name         VARCHAR(50) NOT NULL
);

CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT,
    total        DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

Or with a name for the constraint:

```sql
CONSTRAINT fk_orders_customers
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
```

### What the Foreign Key Protects

```sql
INSERT INTO orders VALUES (103, 99, 500);
-- ERROR: customer_id 99 does not exist in customers
```

You **cannot** create an order for a customer that does not exist.

```sql
DELETE FROM customers WHERE customer_id = 1;
-- ERROR: customer 1 still has orders
```

You **cannot** delete a customer who still has orders (by default).

This protection is called **Referential Integrity**: every reference must point to something real.

### Rules of a Foreign Key

1. The value must **exist** in the parent table's primary key (or be `NULL`).
2. A foreign key column **can repeat** (one customer can have many orders).
3. A foreign key column **can be NULL** (if the relationship is optional).
4. A table can have **more than one** foreign key.

### What Happens When the Parent Row Is Deleted or Changed?

We can choose the behavior with `ON DELETE` and `ON UPDATE`.

```sql
CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT,
    total        DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

| Option | What it does |
|---|---|
| `NO ACTION` / `RESTRICT` | Refuse the delete. This is the default. |
| `CASCADE` | Delete (or update) the child rows too. |
| `SET NULL` | Set the foreign key in child rows to `NULL`. |
| `SET DEFAULT` | Set the foreign key in child rows to its default value. |

**Example:** With `ON DELETE CASCADE`, deleting customer 1 also deletes orders 100 and 102 automatically.

> **Warning for students:** `CASCADE` is powerful. Use it carefully, because it can delete a lot of data.

### Important Order of Work

- **Create** the parent table **first**, then the child table.
- **Insert** into the parent first, then the child.
- **Delete** from the child first, then the parent (unless you use `CASCADE`).
- **Drop** the child table first, then the parent.

---

## 4. Other Table Constraints

### 4.1 `NOT NULL`

The column **must have a value**. It cannot be empty.

```sql
name VARCHAR(50) NOT NULL
```

### 4.2 `UNIQUE`

All values in the column must be **different**. (But unlike a primary key, `UNIQUE` allows NULL in most databases, and a table can have many UNIQUE columns.)

```sql
email VARCHAR(100) UNIQUE
```

### 4.3 `CHECK`

The value must pass a **condition** that we write.

```sql
age    INT CHECK (age >= 18),
salary DECIMAL(10,2) CHECK (salary > 0)
```

### 4.4 `DEFAULT`

If we do not give a value, the database uses a **default value**.

```sql
status       VARCHAR(20) DEFAULT 'active',
created_date DATE DEFAULT CURRENT_DATE
```

---

## 5. Primary Key vs Unique

| | Primary Key | Unique |
|---|---|---|
| Values must be different | Yes | Yes |
| Allows NULL | **No** | Usually yes |
| How many per table | **Only one** | Many |
| Main purpose | Identify each row | Stop duplicate values |

---

## 6. Complete Example

Let us build a small **university** database that uses everything.

```sql
CREATE TABLE departments (
    dept_id    INT PRIMARY KEY,
    dept_name  VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE students (
    student_id  INT PRIMARY KEY,
    name        VARCHAR(50)  NOT NULL,
    email       VARCHAR(100) UNIQUE NOT NULL,
    age         INT CHECK (age >= 16),
    status      VARCHAR(20) DEFAULT 'active',
    dept_id     INT,
    CONSTRAINT fk_students_dept
        FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

### Let us read it line by line

| Column | Constraint | Meaning |
|---|---|---|
| `dept_id` (departments) | `PRIMARY KEY` | Identifies each department |
| `dept_name` | `NOT NULL UNIQUE` | Required, and no two departments with the same name |
| `student_id` | `PRIMARY KEY` | Identifies each student |
| `name` | `NOT NULL` | Name is required |
| `email` | `UNIQUE NOT NULL` | Required, and no repeats |
| `age` | `CHECK (age >= 16)` | Age must be 16 or more |
| `status` | `DEFAULT 'active'` | If not given, it will be `'active'` |
| `dept_id` (students) | `FOREIGN KEY` | Must match a real department |

### Test It

```sql
INSERT INTO departments VALUES (1, 'Computer Science');
INSERT INTO departments VALUES (2, 'Business');

INSERT INTO students (student_id, name, email, age, dept_id)
VALUES (1, 'Ahmed', 'ahmed@mail.com', 20, 1);     -- OK, status becomes 'active'

INSERT INTO students (student_id, name, email, age, dept_id)
VALUES (2, 'Ali', 'ahmed@mail.com', 21, 1);      -- ERROR: email is not unique

INSERT INTO students (student_id, name, email, age, dept_id)
VALUES (3, 'Omar', 'omar@mail.com', 12, 1);       -- ERROR: age check failed

INSERT INTO students (student_id, name, email, age, dept_id)
VALUES (4, 'Mazen', 'mazen@mail.com', 22, 9);       -- ERROR: department 9 does not exist
```

---

## 7. Adding and Removing Constraints Later

We can change a table after we create it using `ALTER TABLE`.

```sql
-- Add a primary key
ALTER TABLE students ADD CONSTRAINT pk_students PRIMARY KEY (student_id);

-- Add a foreign key
ALTER TABLE students
ADD CONSTRAINT fk_students_dept
FOREIGN KEY (dept_id) REFERENCES departments(dept_id);

-- Add a unique constraint
ALTER TABLE students ADD CONSTRAINT uq_students_email UNIQUE (email);

-- Add a check constraint
ALTER TABLE students ADD CONSTRAINT chk_age CHECK (age >= 16);
```

To remove a constraint (the exact command depends on the database):

```sql
-- SQL Server, PostgreSQL, Oracle
ALTER TABLE students DROP CONSTRAINT fk_students_dept;

-- MySQL (for foreign keys)
ALTER TABLE students DROP FOREIGN KEY fk_students_dept;
```

---

## 8. Common Mistakes

1. **Creating the child table before the parent table.** The foreign key has nothing to point to.
2. **Different data types.** The foreign key and the primary key must have the **same data type** (for example, both `INT`).
3. **Using a name or email as a primary key.** These can change or repeat.
4. **Forgetting `NOT NULL`** on columns that are required.
5. **Deleting a parent row first.** Delete the children first, or use `ON DELETE CASCADE`.
6. **Thinking the foreign key must be unique.** It does not. Many orders can belong to one customer.

---

## 9. Quick Summary

| Constraint | Purpose | Example |
|---|---|---|
| `PRIMARY KEY` | Unique ID for each row. Not NULL. One per table. | `student_id INT PRIMARY KEY` |
| `FOREIGN KEY` | Links to a primary key in another table. | `REFERENCES departments(dept_id)` |
| `NOT NULL` | Value is required. | `name VARCHAR(50) NOT NULL` |
| `UNIQUE` | No repeated values. | `email VARCHAR(100) UNIQUE` |
| `CHECK` | Value must pass a condition. | `CHECK (age >= 16)` |
| `DEFAULT` | Value used when none is given. | `DEFAULT 'active'` |

**Remember:**

- **PK** = "Who are you?" (identity of the row)
- **FK** = "Who do you belong to?" (link to another table)
- **Constraints** = "What are the rules?" (protect the data)

---