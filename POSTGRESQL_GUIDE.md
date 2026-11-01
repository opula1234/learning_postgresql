# PostgreSQL Installation and Query Guide

Use this file as a hands-on guide while learning PostgreSQL in this folder. The commands below assume Windows and a local PostgreSQL installation.

## 1. Install PostgreSQL

1. Download the Windows installer from the [official PostgreSQL Windows download page](https://www.postgresql.org/download/windows/).
2. Run the installer and follow its prompts. For a beginner setup, include PostgreSQL Server and command-line tools. pgAdmin is optional and provides a graphical interface.
3. Choose and remember the password for the `postgres` database superuser. Keep it private; do not save it in this repository.
4. Keep the default port, `5432`, unless it is already in use.

## 2. Check the installation

Open PowerShell and run:

```powershell
psql --version
```

If PowerShell says `psql` is not recognized, find the PostgreSQL `bin` folder in the installation directory (often under `C:\Program Files\PostgreSQL\<version>\bin`) and either:

- run `psql.exe` using its full path, or
- add that `bin` folder to your Windows `PATH`, then open a new PowerShell window.

## 3. Connect with `psql`

Connect as the installation's default superuser:

```powershell
psql -U postgres
```

Enter the password when prompted. If the connection needs an explicit host and port:

```powershell
psql -h localhost -p 5432 -U postgres
```

At the `psql` prompt, SQL statements end with a semicolon. These backslash commands are `psql` commands, not SQL:

```text
\conninfo   show connection details
\l          list databases
\dt         list tables in the current schema
\q          quit psql
```

## 4. Create a practice database and table

Run this in `psql` while connected to the default database:

```sql
CREATE DATABASE learning_db;
\c learning_db

CREATE TABLE books (
    id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title text NOT NULL,
    author text,
    published_year integer CHECK (published_year > 0)
);
```

The `\c learning_db` line is a `psql` command. Run SQL commands in a GUI such as pgAdmin without the backslash command; instead, connect the query tool to `learning_db`.

## 5. Add and read data

```sql
INSERT INTO books (title, author, published_year)
VALUES
    ('The Hobbit', 'J. R. R. Tolkien', 1937),
    ('Pride and Prejudice', 'Jane Austen', 1813),
    ('A book with an unknown author', NULL, 2020);

SELECT * FROM books;
```

The `id` column is generated automatically. You can request the inserted row with `RETURNING`:

```sql
INSERT INTO books (title, author, published_year)
VALUES ('Frankenstein', 'Mary Shelley', 1818)
RETURNING id, title;
```

## 6. Query and filter

```sql
-- Choose specific columns
SELECT title, author
FROM books;

-- Filter rows
SELECT title, published_year
FROM books
WHERE published_year < 1950;

-- Sort results
SELECT title, published_year
FROM books
ORDER BY published_year DESC;

-- Search text
SELECT title
FROM books
WHERE title ILIKE '%hobbit%';

-- Test for missing values
SELECT title
FROM books
WHERE author IS NULL;

-- Count rows
SELECT COUNT(*) AS book_count
FROM books;
```

`ILIKE` performs a case-insensitive pattern match in PostgreSQL. `%` matches any sequence of characters.

## 7. Update and delete carefully

Use a `WHERE` clause when you intend to change or remove only selected rows. First run a `SELECT` with the same condition to check which rows match.

```sql
UPDATE books
SET published_year = 1937
WHERE title = 'The Hobbit';

DELETE FROM books
WHERE title = 'A book with an unknown author';
```

An `UPDATE` or `DELETE` without `WHERE` affects every row in the table.

## 8. Use a transaction for a set of changes

```sql
BEGIN;

UPDATE books
SET published_year = 1938
WHERE title = 'The Hobbit';

-- Keep the change:
COMMIT;

-- Or, before committing, undo the transaction instead:
-- ROLLBACK;
```

Choose either `COMMIT` or `ROLLBACK` for the transaction; do not run both expecting the rollback to undo a committed change.

## 9. Inspect and clean up

In `psql`, useful commands include:

```text
\d books      describe the books table
\dt           list tables
\c postgres   connect to the postgres database
\q            exit
```

To remove the practice table or database, use the following only when you are sure you no longer need its data:

```sql
DROP TABLE books;
```

From a different connection, you can drop the practice database with:

```sql
DROP DATABASE learning_db;
```

## 10. Troubleshooting

- **`psql` is not recognized:** Check that PostgreSQL's `bin` directory is on `PATH`, or run `psql.exe` by its full path.
- **Connection refused:** Confirm the PostgreSQL service is running and that the host and port are correct.
- **Password authentication failed:** Check the username and password chosen during setup.
- **Database does not exist:** Connect to an existing database (such as `postgres`) and create `learning_db` first.
- **A command appears stuck at a prompt:** In `psql`, an unfinished quote or missing semicolon can leave the statement incomplete. Press `Ctrl+C` to cancel the current input and try again.

## Learning sequence

1. Practice `SELECT`, `WHERE`, `ORDER BY`, and `LIMIT`.
2. Practice `INSERT`, `UPDATE`, and `DELETE`.
3. Learn constraints, primary keys, and foreign keys.
4. Create a related `authors` table and practice joins.
5. Study grouping, indexes, and transactions.
6. Connect this database to a Python program after you are comfortable with SQL.

Keep expanding this guide with commands you have tested and notes about what they do.
