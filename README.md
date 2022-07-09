## Anomaly-Detection-w-SQL

### Setup PostgreSQL

#### Start Server

```python
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

- Let's connect to the database through user <code>ben</code>
- # has changed to a > (no longer using a Super User account)

```
psql postgres -U ben
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
 
 - Once this is done, you need to add at least one user who has permission to access 
 - the database (aside from the super users, who can access everything)

Commands:

- <code>\list</code> lists all the databases in Postgres
- <code>\connect</code> connect to a database
- <code>\dt</code> list the tables in the currently connected database

```
postgres=> GRANT ALL PRIVILEGES ON DATABASE super_awesome_application TO ben; 
postgres=> \list 
postgres=> \connect super_awesome_application 
postgres=> \dt 
postgres=> \q
```


#### Common Commands

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
