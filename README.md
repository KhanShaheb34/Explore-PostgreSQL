# Explore PostgreSQL

### Brilliant Cloud Research Project

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

- Create a database:
  ```sql
  CREATE DATABASE dbname;
  ```
- Delete/Drop a database:
  ```sql
  DROP DATABASE dbname; # Be careful
  ```
- Connent to a database:
  ```sql
  \c DBNAME     # \c is also used to connect to any other DB, USER, PORT or HOST
  ```

##### Extra:

```
\l  : List of all databases
```

## Tables

- Create a Table

  ```sql
  CREATE TABLE person (
  id BIGSERIAL NOT NULL PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  gender VARCHAR(8) NOT NULL,
  email VARCHAR(150),
  date_of_birth DATE NOT NULL );
  ```

  > [PostgreSQL Data Type Doc](https://www.postgresql.org/docs/12/datatype.html)

- Drop/Delete a Table

  ```sql
  DROP TABLE tablename;
  ```

- Check the Columns of a Table
  ```sql
  \d tablename
  ```

##### Extra:

```
\d  : List of all tables
```

## Add Data

- Insert data into a table

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

- Run SQL from a file

  ```sql
  \i filapath
  ```

  > [Mockaroo](https://www.mockaroo.com/) is a nice website to generate random data

## SELECT

SELECT is used to get data from a table.

- Get everything
  ```sql
  SELECT * FROM tablename;
  ```
- Get particular columns
  ```sql
  SELECT col1, col2 FROM tablename;
  ```
- Get data in sorted order
  ```sql
  SELECT * FROM tablename ORDER BY col_name [ASC/DESC];
  ```
- Get distinct data
  ```sql
  SELECT DISTINCT col_name FROM  tablename;
  ```
- Get data according to condition
  ```sql
  SELECT * FROM tablename WHERE col='value';
  ```
  Multiple conditions can be apllied using `AND`, `OR`
  ```sql
  SELECT * FROM tablename WHERE col1<'value1' AND (col2>='value2' OR col3<>'value3'); # <> is the not equal operator
  ```
