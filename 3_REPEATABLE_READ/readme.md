# Repeatable Read isolation level

Using this level, read phenomena fixed are :
- Dirty Read
- Nonrepeatable read
- Phantom Read

(see [../2_READ_COMMITTED/readme.md](../2_READ_COMMITTED/readme.md) for dirty read phenomenon explanation)

But can occurs :
- Serialization Anomaly

## What is a Nonrepeatable Read ?
Definition is:
> A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).

## Hands on a Nonrepeatable Read fixed, by using Repeatable Read isolation level
<table>
<tr>
<th>Moment</th>
<th>Transaction 1</th>
<th>Transaction 2</th>
<th>Comment</th>
</tr>
<tr>
<td>M1</td>
<td></td>
<td>

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
select * FROM account WHERE name='Germs';
```
</td>
<td>

T2 reads Germs account, amount returned is **100**
</td>
</tr>
<tr>
<td>M2</td>
<td>

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE ACCOUNT SET BALANCE=500 WHERE NAME='Germs';
COMMIT;
```
</td>
<td></td>
<td>
T1 modifies a line (amount 100->500), and commits
</td>
</tr>
<tr>
<td>M3</td>
<td></td>
<td>

```sql
select * FROM account WHERE name='Germs';
```
</td>
<td>

T2 reads Germs account again (in same transaction opened during M1), amount returned is still **100**
</td>
</tr>
<tr>
</table>

## What is a Phantom Read ?
Definition is:
> A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.

So a phantom read is a kind of Nonrepeatable read, but with more or less rows returned than a previous query

## Hands on a Phantom Read fixed, by using Repeatable Read isolation level
<table>
<tr>
<th>Moment</th>
<th>Transaction 1</th>
<th>Transaction 2</th>
<th>Comment</th>
</tr>
<tr>
<td>M1</td>
<td></td>
<td>

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
select * FROM account WHERE name='Germs';
```
</td>
<td>

T2 reads Germs accounts;  only **one row** is returned.
</td>
</tr>
<tr>
<td>M2</td>
<td>

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO ACCOUNT (BALANCE, NAME) VALUES
(20, 'Germs');
COMMIT;
```
</td>
<td></td>
<td>
T1 adds a second account for Germs, and commits.
</td>
</tr>

<tr>
<td>M3</td>
<td></td>
<td>

```sql
select * FROM account WHERE name='Germs';
```
</td>
<td>

T2 reads Germs accounts again (in same transaction opened during M1), **only one row** is sitll returned 
</td>
</tr>
<tr>
</table>

This repeatable read occurs too if we, instead of adding a row (as above), T1 deletes a row ; T2 would see same number of rows as seen before deletion.    
