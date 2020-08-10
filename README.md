<p align="center"><a href="https://www.postgresql.org/">
<img align="center" width="150" src="https://www.postgresql.org/media/img/about/press/elephant.png">
</a></p>

# Explore PostgreSQL

###### Brilliant Cloud Research Project

## Install and Initial Configuration

I'm using Manjaro, so I installed PostgreSQL using `pacman`

```sh
sudo pacman -S postgresql postgis
```

I followed the initial configuration process from [here](https://wiki.archlinux.org/index.php/PostgreSQL#Initial_configuration).

To start PostgreSQL as a service I ran:

```sh
sudo systemctl enable --now postgresql.service
```

To start postgres shell

```sh
sudo -iu postgres
psql
```

or

```sh
psql -U postgres    # Or any other username
```

##### Extra:

```
\q  : Quit psql
\?  : Help
```

## Databases

Create a database:

```sql
CREATE DATABASE dbname;
```

Delete/Drop a database:

```sql
DROP DATABASE dbname; # Be careful
```

Connent to a database:

```sql
\c DBNAME     # \c is also used to connect to any other DB, USER, PORT or HOST
```

##### Extra:

```
\l  : List of all databases
```

## Tables

Create a table

```sql
CREATE TABLE person (
  id BIGSERIAL NOT NULL PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  gender VARCHAR(8) NOT NULL,
  email VARCHAR(150),
  date_of_birth DATE NOT NULL
);
```

> [PostgreSQL Data Type Doc](https://www.postgresql.org/docs/12/datatype.html)

Drop/delete a table

```sql
DROP TABLE tablename;
```

Check the columns of a table

```sql
\d tablename
```

##### Extra:

```
\d  : List of all tables
```

## Add Data

Insert data into a table

```sql
INSERT INTO tablename (
  col_name
)
VALUES ('value');
```

Example:

```sql
INSERT INTO person (
  first_name,
  last_name,
  gender,
  date_of_birth )
VALUES ('Khan', 'Shaheb', 'male', DATE '1234-5-6');
```

Run SQL from a file

```sql
\i filapath
```

> [Mockaroo](https://www.mockaroo.com/) is a nice website to generate random data

## SELECT from table

SELECT is used to get data from a table.

Get everything

```sql
SELECT * FROM tablename;
```

Get particular columns

```sql
SELECT col1, col2 FROM tablename;
```

Get data in sorted order

```sql
SELECT * FROM tablename ORDER BY col_name [ASC/DESC];
```

Get distinct data

```sql
SELECT DISTINCT col_name FROM  tablename;
```

Get data according to condition

```sql
SELECT * FROM tablename WHERE col='value';
```

Multiple conditions can be apllied using `AND`, `OR`

```sql
SELECT * FROM tablename WHERE col1<'value1' AND (col2>='value2' OR col3<>'value3'); # <> is the not equal operator
```

LIMIT and OFFSET

```sql
SELECT * FROM tablename OFFSET 5 LIMIT 5; # To get row from 6 to 10
```

FETCH

```sql
SELECT * FROM tablename OFFSET 5 FETCH FIRST 5 ROW ONLY; # To get row from 6 to 10
```

FETCH is SQL standard but LIMIT isn't.

IN and BETWEEN

```sql
# We can query multiple values for a column like this
SELECT * FROM tablename WHERE col_name='value1' OR col_name='value2' OR col_name='value3';

# Or simply using IN
SELECT * FROM tablename WHERE col_name IN ('value1','value2', 'value3');

# To query values inside a limit
SELECT * FROM tablename WHERE col_name BETWEEN 'value1' AND 'value2';

# Example
SELECT * FROM person WHERE date_of_birth BETWEEN DATE '1990-1-1' AND '2010-1-1';
```

Get data according to a pattern

```sql
# LIKE and ILIKE is used to find pattern data
SELECT * FROM tablename WHERE col_name LIKE 'pattern';

# Such as, to find emails from gmail.com the command will be:
SELECT * FROM person WHERE email LIKE '%gmail.com';

# ILIKE is same as LIKE but case insensitive
```

> Postgres Patterns are almost like Regex. [Full Documentation](https://www.postgresql.org/docs/9.3/functions-matching.html).

Get count of distinct elements in a table

```sql
SELECT col_name, COUNT(*) FROM tablename GROUP BY col_name;

# Add logic to group to show
SELECT col_name, COUNT(*) FROM tablename GROUP BY col_name HAVING COUNT(*) > 10;
```

> More aggregate function is available [here](https://www.postgresql.org/docs/9.5/functions-aggregate.html).

Arithmatic operations

```sql
# Lets calculate a shop item price after 15% vat
SELECT item, price AS original_price, ROUND(price * 0.15, 2) AS vat, price + ROUND(price * 0.15, 2) AS total_price FROM shop;
```

Replace missing data

```sql
# Lets say we want to change every missing age with -1
SELECT COALESCE(age, -1) FROM tablename;
```

## DELETE from table

We can use all of the logics from SELECT to DELET records from the table like this:

```sql
DELETE FROM tablename WHERE col_name='value';
```

> Warning: DELETE command without WHERE will delete entire table.

## UPDATE table

We can use all of the logics from SELECT to UPDATE records from the table like this:

```sql
UPDATE tablename SET col_name='new_value' WHERE id='something';
```

> Warning: UPDATE command without WHERE will update entire table with same data.

## Date and Time

Get current date and time

```sql
SELECT NOW();         # 2020-08-10 21:16:10.516994+06
SELECT NOW()::DATE;   # 2020-08-10
SELECT NOW()::TIME;   # 21:17:09.115756
```

Subtract or add time

```sql
SELECT NOW() - INTERVAL '2 YEAR 1 MONTH';         # 2018-07-10 21:18:32.667185+06
SELECT (NOW() + INTERVAL '2 YEAR 3 MONTH')::DATE; # 2022-11-10
```

Extract data from date

```sql
SELECT EXTRACT(YEAR from NOW());    # 2020
SELECT EXTRACT(MONTH from NOW());   # 8
SELECT EXTRACT(CENTURY from NOW()); # 21
```

Get age from date of birth

```sql
SELECT AGE(NOW(), DATE '2000-7-7') AS age; # 20 years 1 mon 3 days 21:26:26.458119
```

## Primary Key

Primary key is used to uniquely identify records in a table. Two records with same primary key cannot stay in the same table.

To add a primary key to a table.

```sql
ALTER TABLE tablename ADD PRIMARY KEY (col_name);
```

## Constraints

Add other Unique constraint to a table

```sql
ALTER TABLE person ADD CONSTRAINT unique_email UNIQUE(email)
```

We can add a binary constraint using CHECK

```sql
# Such as, to constraint gender to be only Male or Female
ALTER TABLE person ADD CONSTRAINT gender_constraint CHECK(gender = 'Male' OR gender = 'Female');
```

Drop a constraint from a table

```sql
ALTER TABLE person DROP CONSTRAINT constraint_name
```

> Full ducumentation on Constraints is available [here](https://www.postgresql.org/docs/9.4/ddl-constraints.html).

## Handle Exceptions

Inserting a record to a table which has a duplicate from a row that has unique constraint will give error. To handle that error we can do:

```sql
INSERT into tablename (col_name)
VALUES ('values')
ON CONFLICT (unique_col)
DO NOTHING;

# INSERT 0 0
```

This will ignore the insert command and won't give any error.

To update the table with new data `EXCLUDED.col_name` can be used.

```sql
INSERT into tablename (col_name)
VALUES ('values')
ON CONFLICT (unique_col)
DO UPDATE SET col_name=EXCLUDED.col_name;

# INSERT 0 1
```
