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
