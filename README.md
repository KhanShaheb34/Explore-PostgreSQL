<p align="center"><a href="https://www.postgresql.org/">
<img align="center" width="150" src="https://www.postgresql.org/media/img/about/press/elephant.png">
</a></p>

# Explore PostgreSQL

###### Brilliant Cloud Research Project

## Table of Content

- [Install and Initial Configuration](#install-and-initial-configuration)
- [Databases](#databases)
- [Tables](#tables)
- [Add Data](#add-data)
- [SELECT from table](#select-from-table)
- [DELETE from table](#delete-from-table)
- [UPDATE table](#update-table)
- [Date and Time](#date-and-time)
- [Primary Key](#primary-key)
- [Constraints](#constraints)
- [Handle Exceptions](#handle-exceptions)
- [Foreign Key](#foreign-key)
- [Inner Join](#inner-join)
- [Left Join](#left-join)
- [Export Query Results to a CSV File](#export-query-results-to-a-csv-file)
- [Extensions](#extensions)
- [Credits](#credits)
- [Stress-test](/Stress-test/StressTest.ipynb)

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

## Foreign Key

Foreign keys is used to establish a relationship between two tables. Suppose we have a car table:

| Car   |         |              |
| ----- | ------- | ------------ |
| id    | BIGINT  | NOT NULL, PK |
| model | VARCHAR | NOT NULL     |
| price | NUMERIC | NOT NULL     |

and a people table

| People     |         |              |
| ---------- | ------- | ------------ |
| id         | BIGINT  | NOT NULL, PK |
| first_name | VARCHAR | NOT NULL     |
| last_name  | VARCHAR | NOT NULL     |

Let's imagine one person can have atmost car and one car can have only one owner. Then we can add another column to People table that reffer to the records from Car table.

| People     |         |              |
| ---------- | ------- | ------------ |
| id         | BIGINT  | NOT NULL, PK |
| first_name | VARCHAR | NOT NULL     |
| last_name  | VARCHAR | NOT NULL     |
| car_id     | BIGINT  |              |

Here, car_id is a foreign key. To generate the People table we have to write this SQL command:

```sql
CREATE TABLE person (
  id BIGSERIAL NOT NULL PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  car_id BIGINT REFERENCES car (id),
  UNIQUE (car_id)
);
```

Now we can't assign a car_id to a person which isn't available in tha car table.

## Inner Join

Inner join links two table where primary key and foriegn key is found in both table. It is like intersection(∩).

For example if we inner join the person and car table from the previous example. We'll only get the records from person who has a car_id with all of the information from both table.

To inner join them:

```sql
SELECT * FROM person
JOIN car ON person.car_id=car.id;
```

The foreign key should be provided after ON. And we'll get a table with these columns:

```
id, first_name, last_name, car_id, id, model, price
```

If the foreign key and primary key has the same name we can do it like:

```sql
SELECT * FROM person
JOIN car using(car_uid)
```

## Left Join

Left join links the referenced table to the main table. It is like `A + (A ∩ B)` from set.

To left join the person and car table:

```sql
SELECT * FROM person
JOIN car ON car.id=person.car_id;
```

This will also return all of the colums like inner join. But the person records which don't have a car will also be present.

If the foreign key and primary key has the same name we can do it like:

```sql
SELECT * FROM person
LEFT JOIN car using(car_uid)
```

## Export Query Results to a CSV File

We can use `\copy` to export a query results to a file. Such as:

```sql
\copy (SELECT * FROM tablename) TO '/path/to/csv' DELIMITER ',' CSV HEADER;
```

## Extensions

We can chech all of the extensions available in PostgreSQL by using:

```sql
SELECT * FROM pg_available_extensions;
```

To install an extension:

```sql
CREATE EXTENSION IF NOT EXISTS "extension-name";
```

Now we can see available functions by writing `\df`.

---

## Credits

I followed [this video](https://www.youtube.com/watch?v=qw--VYLpxG4) to explore PostgreSQL.

Thanks to **Brilliant Cloud Research** for giving this assignment. 😃

[Table of contents generated with markdown-toc](http://ecotrust-canada.github.io/markdown-toc/).
