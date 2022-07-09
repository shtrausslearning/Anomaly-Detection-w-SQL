## Anomaly-Detection-w-SQL

### Setup PostgreSQL

#### Start Server

```python
pg_ctl -D /usr/local/var/postgres start # start server 
# pg_ctl -D /usr/local/var/postgres stop  # end server
```

- By default, user <code>postgres</code> is created

```
psql postgres
```

- Check which users are installed

```
postgres=# \du
```

- Create additional users, by default the predefined user(s) are <code>super user accounts</code>

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
