# Optimistic lock

Optimistic locking is a way to perform updates on row without usage of implicit not explicit locks on row.
For that, application uses a version column to set and identify versions of row.

It can be practical to avoid lock creation (so contention) and deadlocks because updates operation will fail immediately if version check fails. See after.

## Hands on Optimistic lock, with a failure

### A table with a version column

```sql
CREATE TABLE V_ACCOUNT
(
    id      serial primary key,
    BALANCE NUMERIC,
    NAME    VARCHAR(10),
    VERSION NUMERIC
);
```

### Insert some rows into account table

```sql
TRUNCATE TABLE V_ACCOUNT;
INSERT INTO V_ACCOUNT (BALANCE, NAME, VERSION) VALUES
(100, 'Germs',1),
(200, 'Rihanna',1);
```

## Hands on an optimistic lock failure



<table>
<tr>
<th>Moment</th>
<th>Transaction 1</th>
<th>Transaction 2</th>
<th>Comment</th>
</tr>
<tr>
<td>M1</td>
<td>

```sql
SELECT * FROM V_ACCOUNT WHERE NAME='Germs';
```
</td>
<td>

```sql
SELECT * FROM V_ACCOUNT WHERE NAME='Germs';
```
</td>
<td>

T1 and T2 read Germs account and see current version is **1**.
</td>
</tr>
<tr>

<tr>
<td>M2</td>
<td>

```sql
UPDATE V_ACCOUNT SET BALANCE=500, VERSION = 2 WHERE NAME='Germs' AND VERSION=1;
```
</td>
<td></td>
<td>

T1 modifies Germs account (amount 100->500); it works because version  in where clause is correct.
Now is Germs account is on version **2**. 
</td>
</tr>
<tr>
<td>M3</td>
<td></td>
<td>

```sql
UPDATE V_ACCOUNT SET BALANCE=0, VERSION = 2 WHERE NAME='Germs' AND VERSION=0;
```
<td>

T2 tries to modify Germs account (amount 100->0); it fails because version **1** does not exist anymore in DB.
It fails Silently..... Arrgggg !!
</td>
</tr>
</table>

Notice. Here we did not opened transaction, everything is auto-committed immediately. 
If T1 and T2 use transaction, T2 would fail on M3 only if T1 had committed on M2 (see [../2_READ_COMMITTED/readme.md] for read committed isolation level) 