
---

# ✅ **E-Commerce PostgreSQL Schema (5 Tables + Foreign Keys)**

Below is a realistic minimal e-commerce design covering:

✔ Users
✔ Products
✔ Orders
✔ Order Items
✔ Payments

The schema includes **3 foreign keys** (User → Order, Order → Order Items, Order → Payment).

---

# ✅ **1. users**

Stores all customer accounts.

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Purpose**

* Basic identity of the customer.
* Email kept unique for login / communication.

---

# ✅ **2. products**

List of products available to purchase.

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    description TEXT,
    price NUMERIC(10,2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Purpose**

* Pricing, inventory, catalog details.

---

# ✅ **3. orders**

Every purchase by a customer.

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'PENDING',
    total_amount NUMERIC(12,2) NOT NULL,

    CONSTRAINT fk_user
        FOREIGN KEY (user_id) REFERENCES users(user_id)
        ON DELETE CASCADE
);
```

### **Foreign Key #1**

`user_id → users.user_id`

---

# ✅ **4. order_items**

Line-items belonging to each order.

```sql
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price NUMERIC(10,2) NOT NULL,

    CONSTRAINT fk_order
        FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE,

    CONSTRAINT fk_product
        FOREIGN KEY (product_id) REFERENCES products(product_id)
        ON DELETE RESTRICT
);
```

### **Foreign Key #2**

`order_id → orders.order_id`

### **Foreign Key #3**

`product_id → products.product_id`

---

# ✅ **5. payments**

Tracks payment information for each order.

```sql
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    payment_method VARCHAR(30) NOT NULL,
    amount NUMERIC(12,2) NOT NULL,
    payment_status VARCHAR(20) DEFAULT 'INITIATED',
    paid_at TIMESTAMP,

    CONSTRAINT fk_payment_order
        FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE
);
```

### **Foreign Key #4**

`order_id → orders.order_id`

---

# ⭐ **Diagram (Logical View)**

```
 users (1) ────< orders (N) ────< order_items (N)
                    │
                    └──────< payments (1)
 products (1) ────< order_items (N)
```

---

# 🎯 **Summary**

| Table       | Purpose          | FKs                                      |
| ----------- | ---------------- | ---------------------------------------- |
| users       | Customer details | —                                        |
| products    | Product catalog  | —                                        |
| orders      | Order header     | user_id → users                          |
| order_items | Line items       | order_id → orders, product_id → products |
| payments    | Payment details  | order_id → orders                        |

---

A. 5  JOIN Questions
1️⃣ Fetch all orders with customer name, product name, quantity, order status, and payment status using 4-table JOIN.
2️⃣ List customers who purchased any product priced above ₹1500 using JOIN conditions.
3️⃣ Show the total quantity sold for each product along with the product name using JOIN + GROUP BY.
4️⃣ Find customers who have successful payments, but their orders are still marked as "PENDING".
5️⃣ Identify customers who bought the same product more than once using JOIN + self-referencing logic.
✅ B. 5  SUBQUERY Questions
1️⃣ Find the highest-spending customer using a nested subquery.
2️⃣ List all products that have never been ordered using a NOT IN subquery.
3️⃣ Find customers who placed more orders than the average orders placed per customer.
4️⃣ List all orders whose payment amount is less than the order's total amount (subquery per order).
5️⃣ Find the top 3 most-sold products by quantity using a subquery.
✅ C. 5 GROUP BY + HAVING + WHERE Questions
1️⃣ Show all products whose total revenue (qty × price) is greater than ₹10,000.
2️⃣ List customers who have placed more than 3 orders (use GROUP BY + HAVING).
3️⃣ Find days on which total sales exceeded ₹50,000 (GROUP BY date).
4️⃣ Identify customers whose successful payment percentage is below 50%.
5️⃣ List customers whose total spend across all orders exceeds ₹1,00,000.
