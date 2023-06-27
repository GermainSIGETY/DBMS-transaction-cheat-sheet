# PG Transaction course

Below instructions related to :

A transaction cheat sheet : 
Examples for this course : 

Below, everithing for the practical part

## Install Postgres (docker) and connect to it
```shell
docker run --name some-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=transacc -p 5432:5432 -d  postgres:15.3
```

Use your favorite database client :
- host : localhost
- user : postgres
- password : transacc

## A table for tests 

Create a table:
```sql
CREATE TABLE ACCOUNT
(
    id      serial primary key,
    BALANCE NUMERIC,
    NAME    VARCHAR(10)
);
```

Then Insert some rows into it:
```sql
TRUNCATE TABLE ACCOUNT;
INSERT INTO ACCOUNT (BALANCE, NAME) VALUES
(100, 'Germs'),
(200, 'Rihanna');
```

Here we go for tests !

## Transaction isolation levels 

Pessimistic locking.
See readme.md and SQL files in directories in order to test different isolations levels and see transactions phenomena that can be solved by using them

### [1_READ_UNCOMMITTED/readme.md](1_READ_UNCOMMITTED/readme.md)

Lowest isolation level. Not implemented in Postgres

### [2_READ_COMMITTED/readme.md](2_READ_COMMITTED/readme.md)

Default implementation level in Postgres. It fixes dirty reads phenomenon.

### [3_REPEATABLE_READ/readme.md](3_REPEATABLE_READ/readme.md)

Avoiding Nonrepeatable reads and phantom reads phenomena.

### [4_SERIALIZABLE/readme.md](4_SERIALIZABLE/readme.md)

Avoiding Serialization Anomaly.

## Deadlock

### [5_DEADLOCK/readme.md](5_DEADLOCK/readme.md)

Two transactions produce a deadlock.

## Optimistic locking

### [6_OPTIMISTIC_LOCKING/readme.md](6_OPTIMISTIC_LOCKING/readme.md)

An example of optimistic lock usage, with a failure (Dude your are far too optimistic).