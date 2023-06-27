# Serializable isolation level

Using this level, read phenomena fixed are :
- Dirty Read
- Nonrepeatable read
- Phantom Read
- Serialization Anomaly

(see [2_READ_COMMITTED/readme.md](../2_READ_COMMITTED/readme.md) and [3_REPEATABLE_READ/readme.md](../3_REPEATABLE_READ/readme.md) for other phenomena explanations)


## What is a Serialization anomaly ?
Definition is:
> The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.

To understand this concept, we have to imagine a flow of 2 transactions that both perform write operations, that occur when the two transactions are still opened:
- the two transactions are said serializable if we can execute T1 completely then T2 completely, or T2 completely then T1 completely, and results in DB are identical.
- the two transactions are said not serializable if we cannot execute T1 then T2, or T2 then T1, because those two scenarios produce different results in DB.

Unfortunately, Postgres cannot perform magic trick, if a transaction provoke a serialization anomaly, Postgres will return an error for one of the transaction and the transaction is rollbacked.
It's up to you, valiant developer, to fix this error in your code :)

## Hands on a Serialization anomaly
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
BEGIN TRANSACTION
ISOLATION LEVEL SERIALIZABLE;
UPDATE ACCOUNT SET BALANCE=666
WHERE NAME='Germs';
```
</td>
<td></td>
<td>

T1 updates Germs account with an amount of **666**. Without committing.
</td>
</tr>
<tr>
<td>M2</td>
<td>
</td>
<td>

```sql
BEGIN TRANSACTION
ISOLATION LEVEL SERIALIZABLE;
UPDATE ACCOUNT
SET BALANCE=0 WHERE NAME='Germs';
```
</td>
<td>

T2 updates Germs account with an amount of **0**. Without committing.
</td>
</tr>

<tr>
<td>M3</td>
<td>

```sql
COMMIT;
```
</td>
<td>
</td>
<td>

T1 commits; **ok**
</td>
</tr>
<tr>
<td>M4</td>
<td>
</td>
<td>

```sql
COMMIT;
```
</td>
<td>

T2 commits; **ko**. An Error is returned :
```[40001] ERROR: could not serialize access due to concurrent update``` and transaction T2 is rollbacked.
</td>
</tr>
</table>

Interesting observation there ; it is not the first opened transaction that won (and avoided serialization error), but the first transaction that writes.
