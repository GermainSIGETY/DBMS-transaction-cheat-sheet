# Locks and Deadlocks

Postgres provides multiple features and complexity about locks in order to deal with read/writes conflicts.
Usage of locks can produces deadlocks. We will produce one just after.
For that we will use 'ROW EXCLUSIVE lock' : this lock is an implicit lock used by postgres to avoid conflicts during INSERT, UPDATE, DELETE commands.


Said differently;  INSERT, UPDATE, DELETE commands (commands that modify data), acquire this lock implicitly.

Example ; when a transaction updates a row, it acquires and keep a lock for the row, if another transactions updates the same row it has to wait ; this second updates is blocking.

## What is a deadlock ?
Definition is:
> In concurrent computing, deadlock is any situation in which no member of some group of entities can proceed because each waits for another member, including itself, to take action, such as sending a message or, more commonly, releasing a lock. 

## Hands on a deadlock
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
BEGIN TRANSACTION;
UPDATE ACCOUNT SET BALANCE=0
WHERE NAME='Germs';
```
</td>
<td></td>
<td>

T1 updates Germs account with an amount of **0**. Without committing.
T1 acquires a lock on Germs row.
</td>
</tr>
<tr>
<td>M2</td>
<td>
</td>
<td>

```sql
BEGIN TRANSACTION;
UPDATE ACCOUNT SET BALANCE=1000
WHERE NAME='Rihanna';
```
</td>
<td>

T2 updates Rihanna account with an amount of **1000**. Without committing.
T2 acquires a lock on Rihanna's row.
</td>
</tr>

<tr>
<td>M3</td>
<td>

```sql
UPDATE ACCOUNT SET BALANCE=0
WHERE NAME='Rihanna';
```
</td>
<td>
</td>
<td>

T1 updates Rihanna account with an amount of **1000**.
T1 hangs on T2 lock.
</td>
</tr>
<tr>
<td>M4</td>
<td>
</td>
<td>

```sql
UPDATE ACCOUNT SET BALANCE=1000
WHERE NAME='Germs';
```
</td>
<td>

T2 updates Germs account with an amount of **1000**.
T2 hangs on T1 lock and produce a deadlock. Postgres return an Error ```[40P01] ERROR: deadlock detected Detail:[...]```.
T2 is rollbacked.
</td>
</tr>
<tr>
<td>M5</td>
<td>

```sql
COMMIT;
```
</td>
<td>


</td>
<td>

Because T2 has rollbacked, lock on Rihanna's row has been released, then T1 is unblocked and can commit its transaction ; **ok**.
</td>
</tr>
</table>

Good to know ; The possibility of deadlocks is not affected by isolation levels. Because isolation level changes the behavior of read operations (except serializable), but deadlock occurs due to write operations.