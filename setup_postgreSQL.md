### 1 | Setup PostgreSQL

- First we need to setup <code>PostgreSQL</code>

#### Start Server

```
pg_ctl -D /usr/local/var/postgres start # start server 
# pg_ctl -D /usr/local/var/postgres stop  # end server
```

- By default, a superuser is created, check which users are installed
- Create additional users, by default the predefined user(s) are <code>super user accounts</code>

```
postgres=# \du
```

```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 andrey    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```

- By default, no password is set for the default superuser, let's change the password

```
postgres=# \password andrey
```

- Create a new user, <code>ben</code>, by default the list of roles, attributes will be empty

```
postgres=# CREATE ROLE ben WITH LOGIN PASSWORD 'password'; 
postgres=# \du
```

```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 andrey    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 ben       |                                                            | {}
 ```
 
 - Let's allow the user <code>ben</code> to create databases, using <code>ALTER ROLE</code>

```
postgres=# ALTER ROLE ben CREATEDB; 
postgres=# \du 
```

```
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 andrey    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 ben       | Create DB                                                  | {}
```

- Quit current session in terminal, server keeps on running

```
postgres=> \q
```

#### Connect to Database

- Let's connect to the database through user <code>ben</code>
- has changed to a > (no longer using a Super User account)

```
psql postgres -U ben
```

- Once this is done, you need to add at least one user who has permission to access 
- the database (aside from the super users, who can access everything)

Commands:

- <code>\list</code> lists all the databases in Postgres
- <code>\connect</code> connect to a database
- <code>\dt</code> list the tables in the currently connected database

```
postgres=> GRANT ALL PRIVILEGES ON DATABASE super_awesome_application TO ben; 
postgres=> \list 
```

```
                                  List of databases
           Name            | Owner  | Encoding | Collate | Ctype | Access privileges 
---------------------------+--------+----------+---------+-------+-------------------
 databasename              | andrey | UTF8     | C       | C     | 
 postgres                  | andrey | UTF8     | C       | C     | 
 super_awesome_application | ben    | UTF8     | C       | C     | =Tc/ben          +
                           |        |          |         |       | ben=CTc/ben
 template0                 | andrey | UTF8     | C       | C     | =c/andrey        +
                           |        |          |         |       | andrey=CTc/andrey
 template1                 | andrey | UTF8     | C       | C     | =c/andrey        +
                           |        |          |         |       | andrey=CTc/andrey
 test                      | andrey | UTF8     | C       | C     | 
 ```

- Connect to a particular database <code>super_awesome_application</code>

```
postgres=> \connect super_awesome_application 
```

```
You are now connected to database "super_awesome_application" as user "ben".
```

- When we have some <code>tables</code>, we can call <code>\dt</code>

```
postgres=> \dt 
```

- Now we can create, read, update and delete data with the user <code>ben</code>

### 2 | Upload CSV data to TABLES

- Having created a non-super user, we can upload data to the <code>database</code>

#### Via postgreSQL

- Being connected to our database <code>super_awesome_application=#</code>
- Create <code>table</code> header
- Created <code>TABLES</code> can only be removed by the owners

```
CREATE TABLE DETAILS(emp_id SERIAL,
first_name   VARCHAR(50),
last_name VARCHAR(50),
dob DATE,
city VARCHAR(40));
```

- Copy data to newly created table <code>details</code>

```
COPY details(emp_id,first_name,last_name,dob,city)
FROM '/Users/andrey/Documents/data.txt' 
DELIMITER ','
CSV HEADER;
```

#### Utilising psycopg2

- Utilising <code>psycopg2</code> (via python) may seem a little longer, but we can run it via a script

```python

import psycopg2

DATABASE = 'X'    # database [\list] (which we \connect to)
USER = 'X'        # superused id
PASSWORD = 'X'    # password
HOST = 'XXX.X.X.X' # host ip
PORT = 'XXXX'      # port number  
  
conn = psycopg2.connect(database=f'{DATABASE}',
                        user=f'{USER}', password=f'{PASSWORD}', 
                        host=f'{HOST}', port=f'{PORT}'
)
  
conn.autocommit = True
cursor = conn.cursor()
  
# Create Table Header (SQL query)
sql = '''CREATE TABLE DETAILS(emp_id SERIAL,
first_name   VARCHAR(50),
last_name VARCHAR(50),
dob DATE,
city VARCHAR(40));'''

cursor.execute(sql)
  
# Copy Data into Table (SQL query)
sql2 = '''COPY details(emp_id,first_name,last_name,dob,city)
FROM '/Users/andrey/Documents/data.txt' # absolute path to file
DELIMITER ','
CSV HEADER;'''
  
cursor.execute(sql2)
  
# Fetch the table data (SQL query)
sql3 = '''select * from details;'''
cursor.execute(sql3)
for i in cursor.fetchall():
    print(i)
  
conn.commit()
conn.close()    

```

- Confirm the data has been uploaded via 

```
TABLE details;
```

```
 emp_id | first_name | last_name |    dob     |   city   
--------+------------+-----------+------------+----------
      1 | Max        | Smith     | 2002-02-03 | Sydney
      2 | Karl       | Summers   | 2004-04-10 | Brisbane
      3 | Sam        | Wilde     | 2005-02-06 | Perth
```

#### TABLE OPERATIONS
List databases <code>\list</code> & Connect to databse <code>\connect</code>
- <code>\dt</code> - show tables 
- <code>CREATE TABLE DETAILS((table formats))</code> create table header
- <code>TABLE table_name</code> view table stored in database
- <code>DROP table_name</code> remove table from database 

#### Common SQL Commands 

```
SELECT - extracts data from a database
UPDATE - updates data in a database
DELETE - deletes data from a database
INSERT INTO - inserts new data into a database
CREATE DATABASE - creates a new database
ALTER DATABASE - modifies a database
CREATE TABLE - creates a new table
ALTER TABLE - modifies a table
DROP TABLE - deletes a table
CREATE INDEX - creates an index (search key)
DROP INDEX - deletes an index
```
