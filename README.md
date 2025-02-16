# Automate-your-workflow-with-PostgreSQL-Triggers
Trigger in PostgreSQL is a special kind of function that is automatically executed or fired when certain events occur in a database table. It is used to enforce business rules, track changes, or validate data without requiring manual intervention.

## Why use triggers?
- Automation : Triggers allow you to automate tasks such as updates, inserts, or logging without needing to call them explicitly.
- Consistency : You can maintain consistency in your database by enforcing rules and constraints when certain changes happen.
- Data Integrity : Triggers ensure that no invalid data enters your database, such as automatically setting default values or updating related fields.
- Audit : Triggers help track changes made to data (such as logging insertions or deletions) without relying on manual processes.

## Optimizing performance with triggers
- `Avoiding repeated logic`: Instead of adding manual validation checks in every insert or update query, you centralize that logic in a trigger, making the process more efficient and reducing code redundancy.

- `Reducing the need for application-level checks`: By moving important validation logic to the database layer, you can offload work from your application, which can lead to better performance.

## E-commerce Store Example 🛒
Imagine you have an e-commerce store with two main tables:

`products`: To track inventory.
`sales`: To store the sales data.
`commissions` : To store the commissions data.

```sql
-- Create products table to track inventory
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC,
    stock_quantity INT
);

-- Create sales table to track each sale
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    product_id INT,
    quantity INT,
    salesperson_id INT;
    total_price NUMERIC,
    sale_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- Create commissions table to track commission earned by salespeople
CREATE TABLE commissions (
    id SERIAL PRIMARY KEY,
    sale_id INT,
    salesperson_id INT,
    commission_amount NUMERIC
);
```
## Define the trigger function to update stock and calculate commission

```sql
CREATE OR REPLACE FUNCTION update_stock_and_calculate_commission() 
RETURNS TRIGGER AS $$
BEGIN
    -- Step 1: Update the stock_quantity in the products table
    UPDATE products
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE id = NEW.product_id;

    -- Step 2: Calculate commission (let's say 5% of the sale)
    INSERT INTO commissions (sale_id, salesperson_id, commission_amount)
    VALUES (NEW.id, NEW.salesperson_id, NEW.total_price * 0.05);  -- Using dynamic salesperson_id

    -- Return the NEW row to continue the insert operation in the sales table
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## Create the trigger that fires after an insert into the sales table

```sql
CREATE TRIGGER after_sale_insert
AFTER INSERT ON sales
FOR EACH ROW
EXECUTE FUNCTION update_stock_and_calculate_commission();
```

Now, let’s simulate a sale and see the results!

```sql
-- Insert a product
INSERT INTO products (name, price, stock_quantity) 
VALUES ('Laptop', 1200, 10);
```

| id  | name     | price | stock_quantity |
| --- | -------- | ----- | -------------- |
| 1   | Laptop   | 1200  | 10             | 

```sql
INSERT INTO sales (product_id, quantity, total_price, salesperson_id)
VALUES (1, 2, 2400, 3);
```
Output of product table after sale of 2 laptops

| id  | name     | price | stock_quantity |
| --- | -------- | ----- | -------------- |
| 1   | Laptop   | 1200  | 8              |

Output of commissions table after sale of 2 laptops

| id  | sale_id | salesperson_id | commission_amount |
| --- | ------- | -------------- | ----------------- |
| 1   | 1       | 3              | 120               |

Our trigger automatically inserts a new record into this table with the calculated commission (5% of the total sale price).

### Drop the trigger
- If you want to drop the trigger after_sale_insert on the sales table, you can run

```sql
DROP TRIGGER after_sale_insert ON sales;
```

Dropping a trigger does not delete the trigger function. If you also want to drop the trigger function, you can use

```sql
DROP FUNCTION update_stock_and_calculate_commission();
```



