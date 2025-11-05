Step-by-step hands-on experience building a "shopping cart" system to understand transactions from scratch. You will create tables, run transactions, "play out" fun scenarios, and see exactly how transactions protect your data.

### Step 1: Set Up Your Database

Create the tables in MySQL (or your favorite database):

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price INT
);

CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    wallet INT
);

CREATE TABLE cart (
    user_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY(user_id, product_id)
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    total INT,
    status VARCHAR(20)
);
```
Add sample data:
```sql
INSERT INTO products VALUES (1, 'Pizza', 250), (2, 'Burger', 150), (3, 'Fries', 80);
INSERT INTO users VALUES (1, 'Bunty', 500), (2, 'Shunty', 400);
```

### Step 2: Add Items to the Cart

Let’s say Bunty wants to buy 2 pizzas and 1 fries:
```sql
INSERT INTO cart VALUES (1, 1, 2), (1, 3, 1); -- Bunty's cart, 2 pizzas + 1 fries
```
Check total:
```sql
SELECT SUM(p.price * c.quantity) AS grand_total
FROM cart c
JOIN products p ON p.id = c.product_id
WHERE c.user_id = 1;
```

### Step 3: Make the Purchase—Transaction Time!

Here’s a full transaction script:
```sql
-- Start the transaction!
START TRANSACTION;

-- 1. Check Bunty's Wallet
SELECT wallet FROM users WHERE id = 1;

-- 2. Calculate Cart Total (pretend you did this in your app)
-- Let’s say result is 2*250 + 1*80 = 580

-- 3. Is there enough money?
-- If wallet >= total, keep going!

-- 4. Deduct the money
UPDATE users SET wallet = wallet - 580 WHERE id = 1;

-- 5. Insert the order
INSERT INTO orders (user_id, total, status) VALUES (1, 580, 'confirmed');

-- 6. Clean the cart
DELETE FROM cart WHERE user_id = 1;

-- All good? Save changes!
COMMIT;
```
If anything fails—say, Bunty doesn’t have enough money, or your server crashes after deducting wallet but before placing order—run:
```sql
ROLLBACK;
```
Every change is undone, and Bunty’s wallet and cart are exactly as before.[1]

### Try These Fun Scenarios

- **Scenario 1: Not Enough Money**
    - Set Bunty’s wallet to 500. Try to buy items worth 580. The logic should roll back—no money lost, no wrong order after `ROLLBACK`.

- **Scenario 2: Two Users Shop at the Same Time**
    - Open two SQL terminals. Bunty and Shunty both try to buy the last Pizza.
    - Transactions ensure only one gets it (whoever commits first), preventing double-sold items.

### Play With It: DIY Exercise

1. Change the wallet amounts, cart items, and repeat the process.
2. Simulate a "system crash": run `START TRANSACTION`, update wallet, then close your database session before `COMMIT`. Reconnect—see that nothing was saved![1]
3. Use `ROLLBACK` to practice undoing failed transactions.

### Transaction Safety: Shopping Cart Table

| Step                    | Bunty’s Wallet | Cart Total | Cart Items Left | Order Status     |
|-------------------------|:--------------:|:----------:|:---------------:|:----------------:|
| Before purchase         | 500            | 580        | 2 pizzas, 1 fries | none           |
| After deduction         | -80 (error)    | 580        | unchanged        | none             |
| After rollback          | 500            | 580        | unchanged        | none             |
| After successful commit | -80 (should fail, so skip) | 580 | cleared      | confirmed        |

Transactions make sure you never lose money, double-sell items, or deliver the wrong order.[1]

### BONUS: Isolation Levels Demo

Order items with two users at the same time to see how the system locks the relevant rows, queues up changes, and keeps everything correct—even on a busy Black Friday![1]

***

Try this hands-on, modify the SQL, break things, and experience how transactions protect your shopping cart from chaos. This is your playground to master transactions in the real world.[1]
