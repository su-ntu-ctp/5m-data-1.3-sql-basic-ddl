# Lesson

## Brief

### Preparation

Overview of popular RDBMS and basic understanding of DuckDB as stipulated in [reference](./reference.md).

### Lesson Overview

This lesson introduces the SQL Data Definition Language (DDL) statements. Learners will be able to create a database, schemas and tables, create indexes and constraints (including primary and foreign keys), alter tables, drop tables, create views, insert data, update data, import and export data to/from a database.

---

## Part 1 – Connecting to the Database

You can connect to any database using an SQL client or IDE like [DBeaver](https://dbeaver.io/), which supports databases such as MySQL, PostgreSQL, SQLite, Oracle, DB2, SQL Server, Sybase, Teradata, MongoDB, Cassandra, Redis, and more.

For this lesson, we'll use **DuckDB**, a lightweight, in-process SQL database engine.

---

### 🟢 Use Pre-Created DuckDB File

1. Open **DBeaver**.
2. Use the DuckDB file provided in this repo:  
   ```
   db/unit-1-3.db
   ```
3. Create a new connection to DuckDB. Refer to the steps [here](https://duckdb.org/docs/guides/sql_editors/dbeaver).

> If you expand on the `unit-1-3.db`, you should see a few predefined schemas.  
> - The default schema is `main`.  
> - Any tables you create without specifying a schema will be in `main`.  
> - You can also create additional schemas to organize your tables.

---

### ⚙️ (Optional): Create the DuckDB File Yourself

First, set up the DuckDB file, we need to run the following steps:

1. Create a new conda environment from `environment.yml`
```
  conda env create -f environment.yml
```
2. Activate the conda environment
```
  conda activate ddb
```   
3. Run [create_duckdb.py](./db/create_duckdb.py) to create the database file.
```
  python db/create_duckdb.py
```
Open DBeaver and create a new connection to the DuckDB database file `db/unit-1-3.db`. Refer to the steps [here](https://duckdb.org/docs/guides/sql_editors/dbeaver).

## Part 2 - Executing SQL DDL statements

### 2.1 Create schema

We start by creating a new schema `lesson` to organise our tables.

```sql
CREATE SCHEMA lesson;
```

alternatively, we can also use the following to create a schema if it does not exist yet

```sql
CREATE SCHEMA IF NOT EXISTS lesson;
```

### 2.2 Create table

We will be creating a table `users` in the `lesson` schema. The table will have the following columns:

- `id` - integer
- `name` - varchar
- `email` - varchar

```sql
CREATE TABLE lesson.users (
  id INTEGER,
  name VARCHAR,
  email VARCHAR
);
```

If you just want to create tables in the default (`main`) schema, you can omit the `lesson.` prefix.

### 2.3 Insert data

We can insert data into the table using the `INSERT INTO` statement.

```sql
INSERT INTO lesson.users (id, name, email)
VALUES (1, 'John Doe', 'john.doe@gmail.com');
```

This will insert a new row / record into the `users` table. You can insert multiple rows at once.

```sql
INSERT INTO lesson.users (id, name, email)
VALUES (2, 'Jane Doe', 'jane.doe@gmail.com'),
       (3, 'John Smith', 'john.smith@gmail.com');
```

> Insert two more rows with contiguously increasing `id` values, random `name`s and `email`s.

### 2.4 Drop table

We can drop (remove) the table using the `DROP TABLE` statement.

```sql
DROP TABLE lesson.users;
```

## Part 3 - Executing SQL DDL statements based on ERD

Let's use a similar case study (scenario) from the previous lesson.

In a school system whose classes have students and teachers. Each student belongs to a single class. Each teacher may teach more than one class, but each class only has one teacher.

Refer to the ERD below.

```dbml
Table students {
  id int [pk]
  name varchar
  address varchar
  phone varchar
  email varchar
  class_id int
}

Table teachers {
  id int [pk]
  name varchar
  age int
  address varchar
  phone varchar
  email varchar
}

Table classes {
  id int [pk]
  name varchar
  teacher_id int
}

Ref: students.class_id > classes.id // many-to-one

Ref: classes.teacher_id > teachers.id // many-to-one
```

### 3.1 Create tables with constraints

We will be creating the 3 tables `students`, `teachers` and `classes` in the `lesson` schema. How do we create the primary and foreign keys? By using a concept called constraints.

#### Constraints

Constraints are used to specify rules for data in a table. We use constraints mainly for the following purposes:

- `primary key` or `unique` define a column, or set of columns, that are a unique identifier for a row in the table.
  - primary key constraints and unique constraints are identical except that a table can only have one primary key constraint defined, but many unique constraints and a primary key constraint also enforces the keys to not be NULL.
- `foreign key` defines a column, or set of columns, that refer to a primary key or unique constraint from another table. The constraint enforces that the key exists in the other table.
- `not null` specifies that the column cannot contain any NULL values. By default, all columns in tables are nullable.
- `default` specifies a default value for the column when no value is specified.
- `check` constraint allows you to specify an arbitrary boolean expression. Any columns that do not satisfy this expression violate the constraint.

```sql
CREATE TABLE lesson.teachers (
  id INTEGER PRIMARY KEY, -- primary key
  name VARCHAR NOT NULL, -- not null
  age INTEGER CHECK(age > 18 AND age < 70), -- check
  address VARCHAR,
  phone VARCHAR,
  email VARCHAR CHECK(CONTAINS(email, '@')) -- check
);

CREATE TABLE lesson.classes (
  id INTEGER PRIMARY KEY, -- primary key
  name VARCHAR NOT NULL, -- not null
  teacher_id INTEGER REFERENCES lesson.teachers(id) -- foreign key
);
```

> Complete the `CREATE TABLE` statement for the `students` table.

### 3.2 Create indexes

Indexes are used to improve the performance of queries. They are not required but are recommended for tables with many rows. They are used to _retrieve data from the database more quickly than otherwise_. Indexes are created using one or more columns of a database table. The users cannot see the indexes, they are just used to speed up searches/queries.

```sql
-- Create a unique index 'teachers_name_idx' on the column name of table teachers.
CREATE UNIQUE INDEX teachers_name_idx ON lesson.teachers(name);
-- Create index 'students_name_idx' that allows for duplicate values on the column name of table students.
CREATE INDEX students_name_idx ON lesson.students(name);
```

### 3.3 Alter tables

We can alter the tables to add, rename or remove columns.

Add column 'start_date' to table classes.

```sql
ALTER TABLE lesson.classes ADD COLUMN start_date DATE;
```

Rename column 'name' to 'code' in table classes.

```sql
ALTER TABLE lesson.classes RENAME name TO code;
```

## Part 4 - Tables vs Views

Tables and views are both ways to store data. What is the difference between them? A table is a physical copy of the data (materialized), while a view is a virtual copy of the data. A view is a query that is run on the fly when you access the view. A view is not stored in the database, but the query that defines the view is stored in the database.

### 4.1 Create view

We will be creating a view `students_view` in the `lesson` schema. The view will have the following columns:

- `id` - integer
- `name` - varchar
- `email` - varchar

```sql
CREATE VIEW lesson.students_view AS
SELECT id, name, email
FROM lesson.students;
```

> Create a view `teachers_view` with the same columns as `students_view` but for the `teachers` table.

We will learn more about the syntax in the next lesson.

## Part 5 - Importing / exporting data

### 5.1 Importing data

We can import data from a CSV file into a table.

```sql
COPY table_name FROM 'file_name.csv' (AUTO_DETECT TRUE);
```

Open [data/import.sql](./data/import.sql) which is the SQL script that contains the import data statements for the 3 tables. Prepend the full directory path to the CSV files, e.g. `/path/to/directory/5m-data-1.3-sql-basic-ddl/data/teachers.csv`

### 5.2 Updating data

We can update the data in the table using the `UPDATE` statement.

Let's say `Linda Garcia` changed her email to `linda.g@example.com`, we can update the `email` of the student with id 4 (her id) to `linda.g@example.com`. The `WHERE` clause is used to specify which rows to update.

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE id = 4;
```

> Try and update the email of `Linda Garcia` using her name instead of her id as the condition.

### 5.3 Exporting data

Let's export the data from the student table into a CSV file delimited with `|`. Remember to prepend the full directory path to the CSV file.

```sql
COPY (SELECT * FROM lesson.students) TO 'students_new.csv' WITH (HEADER 1, DELIMITER '|');
```

We can also export the data into a JSON file (you will learn more about JSON in Module 2).

```sql
COPY (SELECT * FROM lesson.students) TO 'students.json';
```

> Repeat the above steps for the `teachers` table.
