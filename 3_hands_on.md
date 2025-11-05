End-to-end hands-on process for handling the "insufficient funds" transaction using Python with SQL (works with MySQL, PostgreSQL, etc.). This approach combines user input, fetching balances, making business logic checks, and ensuring safe transactions using `COMMIT`/`ROLLBACK`.

***

### Step 1: Setup (Python + SQL connector)

First, install your connector. For MySQL:
```bash
pip install mysql-connector-python
```

### Step 2: Sample Database Tables

Suppose you have these tables:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    wallet INT
);

CREATE TABLE cart (
    user_id INT,
    product_id INT,
    quantity INT
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price INT
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    total INT,
    status VARCHAR(20)
);
```

### Step 3: Python Code for the Transaction

```python
import mysql.connector

# Connect to your database (change details as required)
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="your_password",
    database="shop"
)
cursor = conn.cursor()

user_id = 1  # Bunty

try:
    # Start transaction
    conn.start_transaction()

    # 1. Get cart total (sum up all cart items for Bunty)
    cursor.execute("""
    SELECT SUM(p.price * c.quantity) AS grand_total
    FROM cart c
    JOIN products p ON p.id = c.product_id
    WHERE c.user_id = %s
    """, (user_id,))
    result = cursor.fetchone()
    grand_total = result[0] if result[0] else 0

    # 2. Get Bunty's wallet balance
    cursor.execute("SELECT wallet FROM users WHERE id = %s", (user_id,))
    bunty_wallet = cursor.fetchone()[0]

    # 3. Business logic & actions
    if grand_total > bunty_wallet:
        # Not enough money! Rollback and exit.
        print("Insufficient funds. No purchase made.")
        conn.rollback()
    else:
        # Enough money: deduct wallet, place order, clear cart
        cursor.execute("UPDATE users SET wallet = wallet - %s WHERE id = %s", (grand_total, user_id))
        cursor.execute("INSERT INTO orders (user_id, total, status) VALUES (%s, %s, %s)", (user_id, grand_total, 'confirmed'))
        cursor.execute("DELETE FROM cart WHERE user_id = %s", (user_id,))
        conn.commit()
        print("Purchase complete!")

except Exception as e:
    print("Something went wrong:", e)
    conn.rollback()
finally:
    cursor.close()
    conn.close()
```

### What Each Step Does

- Calculates total cost of items in the user's cart.
- Fetches current wallet balance.
- Checks business rule: if not enough funds, calls `rollback()` so *nothing* changes in the database.
- If enough funds, deducts from wallet, creates an order, and empties the cart—all in one atomic transaction.
- Catches any errors (`Exception`) and rolls back if something goes wrong mid-way.

***

### Fun Exercise to Try

- Add/remove items in the cart, alter wallet balances, and test different `grand_total` values.
- Simulate "failure" by purposely raising an error in the middle and see how rollback keeps your data safe and consistent.

You'll see the magic of transactions at work, just like a real-world shopping cart scenario!
