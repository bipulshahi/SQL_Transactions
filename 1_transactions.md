A transaction in databases is a set of actions treated as a single, indivisible unit—either all actions succeed, or none do, so your data is always safe and consistent.

### What Is a Transaction?
A transaction is like a promise your database makes: if you ask it to do many related things together (like send money from one account to another), it will do all of them or none of them at all. Think of shopping online: adding items to your cart, paying, and getting a confirmation—if something fails, you don’t want your money taken but no order placed.

### Fun, Easy Example
Imagine transferring ₹500 from Ravi to his friend:
- The bank needs to (1) deduct ₹500 from Ravi and (2) add ₹500 to his friend.
- Both steps **must** happen, or neither should, so you never lose or magically create money.
- If the server crashes after deducting but before crediting, a transaction ensures this never happens—it “rolls back” so both accounts stay correct.

Here’s how it looks:

| Step                | Ravi’s Account | Friend’s Account |
|---------------------|:--------------:|:----------------:|
| Before              | 1000           | 500              |
| After Deduct        | 500            | 500              |
| After Credit        | 500            | 1000             |

If a crash happens after step 1, the system undoes everything, as if nothing happened.

### ACID Properties: What Makes Transactions Special?
- **Atomicity:** All-or-nothing; either every part succeeds, or everything is canceled.
- **Consistency:** The database stays correct, following rules like “no negative balances”.
- **Isolation:** Transactions don’t interfere; even if two people buy things at the same time, both get correct results.
- **Durability:** Once done, it stays done, even with a power failure.

### The Hands-On: Try Transactions in MySQL

**Let’s play bank owner:**

Suppose you have these accounts:

| id  | name  | balance |
|-----|-------|---------|
| 101 | Ravi  | 1000    |
| 102 | Friend| 500     |

**Move ₹100 from Ravi (101) to Friend (102):**
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 101;
UPDATE accounts SET balance = balance + 100 WHERE id = 102;
COMMIT;
```
If something fails before `COMMIT` (e.g., a power cut), run:
```sql
ROLLBACK;
```
This cancels everything and puts both accounts back to their starting point—like magic!

**YOU TRY THIS:**
1. Create a table with sample balances.
2. Use `START TRANSACTION` in SQL, change balances as above.
3. Before `COMMIT`, disconnect or close your SQL window. Reconnect and see: nothing changed!
4. Redo with `COMMIT`—data updates only if it’s fully successful.

### Another Fun Example: Online Pizza Party

Imagine Bunty and Shunty both want to buy food from the same wallet at the same time:
- If both transactions aren’t isolated, they might overspend, or one’s payment disappears.
- With correct isolation, Bunty’s payment finishes first, then Shunty’s uses the new balance—no mix-ups!

Try running two transactions simultaneously (on different SQL terminals) and see how isolation levels affect the result.

### Key Takeaways
- Transactions are the “safety net” for databases; they keep your data right, even when things go wrong.
- The magic is in “ACID”—Atomic, Consistent, Isolated, Durable.
- Practicing with sample tables, `START TRANSACTION`, `COMMIT`, and `ROLLBACK` in your own database makes these ideas real.
