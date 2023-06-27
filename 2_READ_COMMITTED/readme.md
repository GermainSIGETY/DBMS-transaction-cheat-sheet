# Read Committed isolation level

Default isolation level in Postgres. with this level, read phenomenon fixed is :
- Dirty Read

But can occur :
- Nonrepeatable Read
- Phantom Read
- Serialization Anomaly


## What is a dirty read ?
Definition is:
> A transaction reads data written by a concurrent uncommitted transaction.

## Hands on a dirty read fixed, by using Read Committed isolation level
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
ISOLATION LEVEL READ COMMITTED;
UPDATE ACCOUNT SET BALANCE=500
WHERE NAME='Germs';
```
</td>
<td></td>
<td>
T1 modifies a line (amount 100->500) without committing it.
</td>
</tr>
<tr>
<td>M2</td>
<td></td>
<td>

```sql
BEGIN TRANSACTION 
ISOLATION LEVEL READ COMMITTED;
select * FROM account
WHERE name='Germs';
```
<td>

T2 reads Germs account, amount returned is still **100**.
</td>
</tr>
<tr>
<td>M3</td>
<td>

```sql
COMMIT;
```
</td>
<td></td>
<td></td>
</tr>
<tr>
<td>M4</td>
<td></td>
<td>

```sql
select * FROM account
WHERE name='Germs';
```
</td>
<td>

T2 reads Germs account, amount returned is now **500**.
</td>
</tr>
</table>
