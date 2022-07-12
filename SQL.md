#### 1 | SQL data types

- Exact, approximate & time data types in SQL

- Exact numerical data types
    - int	-2,147,483,648	2,147,483,647
    - bigint	-9,223,372,036,854,775,808	9,223,372,036,854,775,807
    - smallint	-32,768	32,767
    - tinyint	0	255
    - bit	0	1
    - decimal	-10^38 +1	10^38 -1
    - numeric	-10^38 +1	10^38 -1

- Approximate numerical data types
    - float	-1.79E + 30	1.79E + 308
    - real	-3.40E + 38	3.40E + 38

- Date & Time
    - datetime	Jan 1, 1753	Dec 31, 9999
    - smalldatetime	Jan 1, 1900	Jun 6, 2079
    - date	Stores a date like June 30, 1991	-
    - time	Stores a time like 12:30 P.M.	

#### 2 | SQL Constraint 

- <code>NOT NULL</code>
    - Ensures that a column cannot have a NULL value
- <code>DEFAULT</code>
    - Provide a default value in a column when none is specified
- <code>UNIQUE</code> 
    - Ensures that all values in a column are unique 
- <code>PRIMARY</code>
    - A primary key is a field that uniquely identifies each row/record in a database table
- <code>FOREIGN</code>
    - Uniquely identifies a row/record in any other database table. A foreign key is a key used to link two tables together
- <code>CHECK</code>
    - Ensures that all values in a column satisfy certain conditions   

#### 3 | CREATE, DROP & USE DATABASE

- Create, remove & use database (make active)

```sql
CREATE DATABASE database;
DROP DATABASEA database;
SHOW DATABASES;
USE testDB1; 
```

#### 4 | CREATE, DROP & USE TABLE

- Create table header, describe table & drop table

```sql
CREATE TABLE CUSTOMERS(
  ID   INT              NOT NULL,
  NAME VARCHAR (20)     NOT NULL,
  AGE  INT              NOT NULL,
  ADDRESS  CHAR (25) ,
  SALARY   DECIMAL (18, 2),  /* The (18,2) simply means that we can have 18 digits with 2 of them after decimal point*/
  PRIMARY KEY (ID)
);

DESC CUSTOMERS;

DROP TABLE CUSTOMERS;

DESC CUSTOMERS;
);
```

```
Field	Type	Null	Key	Default	Extra
ID	int(11)	NO	PRI	NULL	
NAME	varchar(20)	NO		NULL	
AGE	int(11)	NO		NULL	
ADDRESS	char(25)	YES		NULL	
SALARY	decimal(18,2)	YES		NULL	
```

#### 4 | INSERT INTO 

```sql
CREATE TABLE CUSTOMERS(
  ID   INT              NOT NULL,
  NAME VARCHAR (20)     NOT NULL,
  AGE  INT              NOT NULL,
  ADDRESS  CHAR (25) ,
  SALARY   DECIMAL (18, 2),  /* The (18,2) simply means that we can have 18 digits with 2 of them after decimal point*/
  PRIMARY KEY (ID)
);

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (1, 'Mark', 32, 'Texas', 50000.00 );

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (2, 'John', 25, 'NY', 65000.00 );

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (3, 'Emily', 23, 'Ohio', 20000.00 );

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (4, 'Bill', 25, 'Chicago', 75000.00 );

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (5, 'Tom', 27, 'Washington', 35000.00 );

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (6, 'Jane', 22, 'Texas', 45000.00 );
```

#### 5 | SELECT CLAUSE

```sql
CREATE TABLE CUSTOMERS(
  ID   INT              NOT NULL,
  NAME VARCHAR (20)     NOT NULL,
  AGE  INT              NOT NULL,
  ADDRESS  CHAR (25) ,
  SALARY   DECIMAL (18, 2),  /* The (18,2) simply means that we can have 18 digits with 2 of them after decimal point*/
  PRIMARY KEY (ID)
);

INSERT INTO CUSTOMERS (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (1, 'Mark', 32, 'Texas', 50000.00 ),
       (2, 'John', 25, 'NY', 65000.00 ),
       (3, 'Emily', 23, 'Ohio', 20000.00 ),
       (4, 'Bill', 25, 'Chicago', 75000.00 ),
       (5, 'Tom', 27, 'Washington', 35000.00 ),
       (6, 'Jane', 22, 'Texas', 45000.00 );

SELECT ID, NAME, SALARY FROM CUSTOMERS;
```

#### 6 | Aggregate Functions in SQL

- <code>count</code>, <code>sum</code>, <code>avg</code>, <code>min</code>, <code>max</code>

```sql
SELECT count(rows);
FROM table;
```
