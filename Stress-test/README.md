# PostgreSQL Stress Test

This is the markdown version of [this notebook](StressTest.ipynb)

## Data

At first we need a lot of data to do the stress test. I tried to use mockaroo to generate a lot of data. But they provide only 1000 rows of data to free user.

So, I downloaded 1000 rows of data and wrote this script to generate a million of random data from those 1000 rows.


```python
import pandas as pd
from random import randint

def generate_million_rows(input_path, output_path):
    mockData = pd.read_csv(input_path)

    million_data = {
        "first_name": [],
        "last_name": [],
        "email": [],
        "gender": [],
        "country": [],
        "dob": [],
    }


    for i in range(1_000_000):
        for col in million_data:
            million_data[col].append(mockData[col][randint(0, 999)])

    million_dataframe = pd.DataFrame(million_data)
    million_dataframe.to_csv(output_path, index=False, header=False)
    
generate_million_rows('MOCK_DATA.csv', 'MILLION_MOCK_DATA.csv')
```

## Goal

I'm going to use `psycopg2` to connect PostgreSQL with my python application and I will check the time of every instruction to the database using the `time` package.


```python
import psycopg2
import time
```

## Connect to Database

I'll work on my "stress" database.


```python
con = psycopg2.connect(database="stress", user="postgres",
                       password="", host="127.0.0.1", port="5432")
cur = con.cursor()
print("Connected to Database")
```

    Connected to Database


## Create Table

Let's create a function to measure the runtime.


```python
def measureTimeOf(fn, args=None, verbose=True):
    start_time = time.time()
    if(args):
        out = fn(args)
    else:
        out = fn()
    run_time = time.time() - start_time
    if verbose:
        print("Time: %ss" % run_time)
    return run_time, out
```

Now I'll create a table and measure how much time it takes.


```python
def createPersonTable():
    cur.execute('''CREATE TABLE person (
                    id BIGSERIAL PRIMARY KEY NOT NULL,
                    first_name VARCHAR(50) NOT NULL,
                    last_name VARCHAR(50) NOT NULL,
                    email VARCHAR(150) NOT NULL,
                    gender VARCHAR(10) NOT NULL,
                    country VARCHAR(50) NOT NULL,
                    dob DATE NOT NULL
                );''')
    con.commit()

measureTimeOf(createPersonTable)
print("Table Created")
```

    Time: 0.1529533863067627s
    Table Created


So, it took only 0.153s to create a table.

## InsertData

Now we will insert 1,000,000 rows that we generated before and measure the time.


```python
def insertOneMillionRows(data_path):
    with open(data_path) as data:
        cur.copy_from(data, 'person', columns=(
            "first_name", "last_name", "email", "gender", "country", "dob"), sep=",")
    con.commit()
    
insertion_time, _ = measureTimeOf(insertOneMillionRows, "MILLION_MOCK_DATA.csv")
print("One million rows inserted to the table.")
```

    Time: 6.7815399169921875s
    One million rows inserted to the table.


We can see that, it took 6.7815 seconds to insert one million rows to the database. I used `copy_from` instead of SQL `INSERT into`. Beacause inserting a single row at a time is much slower.

Now I'll insert another 4 million rows and measure time taken to insert every million. Then we will check the total time to insert all 5 million rows.

But to insert 4 million rows, we have to generate the data first.


```python
for i in range(4):
    generate_million_rows('MOCK_DATA.csv', 'MILLION_MOCK_DATA_'+str(i)+'.csv')
```


```python
total_time = insertion_time

for i in range(4):
    print("Already loaded "+str(i + 1)+" million rows. Loading another one million...")
    insertion_time, _ = measureTimeOf(insertOneMillionRows, 'MILLION_MOCK_DATA_'+str(i)+'.csv')
    total_time += insertion_time
    print("\n")

print("Total time taken to load 5M rows: %ss" % total_time)
```

    Already loaded 1 million rows. Loading another one million...
    Time: 5.958866357803345s
    
    
    Already loaded 2 million rows. Loading another one million...
    Time: 5.923978090286255s
    
    
    Already loaded 3 million rows. Loading another one million...
    Time: 8.954365968704224s
    
    
    Already loaded 4 million rows. Loading another one million...
    Time: 6.144176721572876s
    
    
    Total time taken to load 5M rows: 33.76292705535889s


## Query

Five million rows is already inserted to the database. Now, it's time to check how many seconds it takes to query something from the database.

Let's start with a simple query.


```python
def selectAllFromPersonWhere(query):
    select_query = "SELECT * FROM person WHERE " + query
    cur.execute(select_query)
    data = cur.fetchall()
    con.commit()
    return data

runtime, records = measureTimeOf(selectAllFromPersonWhere, "id BETWEEN 100 AND 103;")

for row in records:
    print(records)
```

    Time: 0.18014740943908691s
    [(100, 'Lu', 'Embury', 'wstpierre6o@japanpost.jp', 'Male', 'Thailand', datetime.date(2005, 1, 24)), (101, 'Dorena', 'Murtell', 'jneylone0@patch.com', 'Male', 'Croatia', datetime.date(1956, 7, 11)), (102, 'Gerome', 'Guly', 'wambridge4o@com.com', 'Male', 'Czech Republic', datetime.date(1980, 12, 14)), (103, 'Cherish', 'Broom', 'gtumeltygp@addtoany.com', 'Male', 'Czech Republic', datetime.date(2002, 8, 24))]
    [(100, 'Lu', 'Embury', 'wstpierre6o@japanpost.jp', 'Male', 'Thailand', datetime.date(2005, 1, 24)), (101, 'Dorena', 'Murtell', 'jneylone0@patch.com', 'Male', 'Croatia', datetime.date(1956, 7, 11)), (102, 'Gerome', 'Guly', 'wambridge4o@com.com', 'Male', 'Czech Republic', datetime.date(1980, 12, 14)), (103, 'Cherish', 'Broom', 'gtumeltygp@addtoany.com', 'Male', 'Czech Republic', datetime.date(2002, 8, 24))]
    [(100, 'Lu', 'Embury', 'wstpierre6o@japanpost.jp', 'Male', 'Thailand', datetime.date(2005, 1, 24)), (101, 'Dorena', 'Murtell', 'jneylone0@patch.com', 'Male', 'Croatia', datetime.date(1956, 7, 11)), (102, 'Gerome', 'Guly', 'wambridge4o@com.com', 'Male', 'Czech Republic', datetime.date(1980, 12, 14)), (103, 'Cherish', 'Broom', 'gtumeltygp@addtoany.com', 'Male', 'Czech Republic', datetime.date(2002, 8, 24))]
    [(100, 'Lu', 'Embury', 'wstpierre6o@japanpost.jp', 'Male', 'Thailand', datetime.date(2005, 1, 24)), (101, 'Dorena', 'Murtell', 'jneylone0@patch.com', 'Male', 'Croatia', datetime.date(1956, 7, 11)), (102, 'Gerome', 'Guly', 'wambridge4o@com.com', 'Male', 'Czech Republic', datetime.date(1980, 12, 14)), (103, 'Cherish', 'Broom', 'gtumeltygp@addtoany.com', 'Male', 'Czech Republic', datetime.date(2002, 8, 24))]


It takes 0.18s to run a simple query. Now, I'll run 100,000 queries check how much time it takes.


```python
total_time = 0.0

for i in range(100_000):
    random_id = randint(0, 4999999)
    runtime, records = measureTimeOf(selectAllFromPersonWhere, "id=%d;" % random_id, verbose=False)
    total_time += runtime

print("100,000 simple queries took %s seconds" % total_time)
```

    100,000 simple queries took 18.64275074005127 seconds


## Delete

Time to delete some data and check how much time it takes. Let's act like Thanos and delete half of the data to bring perfect balance. 😎


```python
def deleteFromPersonWhere(query):
    select_query = "DELETE FROM person WHERE " + query
    cur.execute(select_query)
    con.commit()

measureTimeOf(deleteFromPersonWhere, "id%2=0;")
print("Deleted all rows with even ID.")
```

    Time: 30.080256462097168s
    Deleted all rows with even ID.


So, it took 30.08 seconds to delete half od the rows.

## Drop Table

Time to drop the whole table with 2.5M rows and check how much time does it take.


```python
def dropPersonTable():
    drop_query = "DROP TABLE person"
    cur.execute(drop_query)
    con.commit()

measureTimeOf(dropPersonTable)
print("Droped the person table")
```

    Time: 0.5276706218719482s
    Droped the person table


## Conclusion

Times taken:
    
* Create table: 0.153s
* 1,000,000 row insert: 6.782s
* 5,000,000 row insert: 33.763s
* 100,000 queries: 18.643s
* 2,500,000 row delete: 30.08s
* Drop table with 2,500,000 rows: 0.528s

## My Specs

```
OS: Manjaro Linux x86_64 
CPU: Intel i5-7200U (4) @ 3.100GHz 
GPU: NVIDIA GeForce 930MX 
GPU: Intel HD Graphics 620 
Memory: 7638MiB
```
