# Part 2: Advanced Java Knowledge for Swing

# Lesson 18: Advanced SQL + JDBC Optimization

## Designing High Performance Café POS Database Access

### (Java 25 + MySQL + Enterprise Architecture)

ဒီ Lesson မှာ Java Code တင်မဟုတ်ဘဲ **Database Performance Engineering** ကို လေ့လာပါမယ်။

Production Café POS System မှာ:

- Products 100,000+
    
- Sales Records Millions
    
- Multiple Cashier Terminals
    
- Real-time Inventory
    

ရှိလာရင် Simple JDBC CRUD နဲ့ မလုံလောက်တော့ပါဘူး။

---

# 1. Why Database Optimization Matters?

Small Database:

```
products = 100 rows

SELECT * FROM products

0.01 sec

```

---

Large Database:

```
products = 10,000,000 rows

SELECT * FROM products

???

```

Problem:

- Slow search
    
- High CPU
    
- Memory usage
    
- User waiting time
    

---

# 2. Database Query Execution

When you run:

```sql
SELECT *
FROM products
WHERE name='Coffee';

```

MySQL does:

```
SQL Query

    |
    v

Parser

    |
    v

Optimizer

    |
    v

Execution Plan

    |
    v

Storage Engine

    |
    v

Result

```

---

Database Optimization means:

> Help Database find data faster.

---

# 3. Index Concept

Index ဆိုတာ:

> Database table ကို search မြန်အောင် ပြုလုပ်ထားတဲ့ special data structure ဖြစ်ပါတယ်။

Book Index နဲ့တူပါတယ်။

---

Without Index:

```
products table


1 Coffee
2 Tea
3 Cake
...
10000000 Juice


Search Coffee


Check row 1
Check row 2
Check row 3
...
Check row 10000000

```

---

With Index:

```
Index

Coffee -> Row 1

Cake   -> Row 3

Tea    -> Row 2


Direct jump

```

---

# 4. Creating Index

Example:

```sql
CREATE INDEX idx_product_name
ON products(name);

```

---

Before:

```sql
SELECT *
FROM products
WHERE name='Coffee';

```

Slow.

---

After:

```sql
Index Search

      |

Coffee row

```

Fast.

---

# 5. Primary Key Index

Example:

```sql
CREATE TABLE products(

id INT PRIMARY KEY,

name VARCHAR(100),

price DECIMAL(10,2)

);

```

---

MySQL automatically creates index:

```
PRIMARY INDEX

id

```

---

Search:

```sql
SELECT *
FROM products
WHERE id=100;

```

Very fast.

---

# 6. Composite Index

Example:

Sales search:

```sql
SELECT *
FROM orders

WHERE customer_id=10

AND order_date='2026-07-31';

```

---

Create:

```sql
CREATE INDEX idx_order_search

ON orders(
customer_id,
order_date
);

```

---

Important:

Order matters.

Good:

```sql
customer_id,
order_date

```

Because query uses both.

---

# 7. Index Overuse Problem

Too many indexes:

```
products

 |
 +-- name index
 |
 +-- price index
 |
 +-- category index
 |
 +-- date index

```

Problems:

- Insert slower
    
- Update slower
    
- More storage
    

---

Rule:

Index columns used for:

✅ Search

✅ Join

✅ Sorting

---

# 8. EXPLAIN Query Plan

Before optimization:

```sql
EXPLAIN

SELECT *
FROM products
WHERE name='Coffee';

```

Example:

```
type: ALL

rows: 1000000

```

Meaning:

Full table scan.

---

After index:

```
type: ref

rows: 1

```

Much faster.

---

# 9. Pagination

Problem:

Product list:

```sql
SELECT *
FROM products;

```

Returns:

```
1 million rows

```

Bad.

---

Solution:

Pagination.

Example:

```sql
SELECT *

FROM products

LIMIT 50

OFFSET 0;

```

---

Page 1:

```
Rows 1-50

```

---

Page 2:

```sql
LIMIT 50 OFFSET 50

```

---

# 10. JDBC Pagination Example

Repository:

```java
public List<Product> findPage(
int page,
int size
){

String sql =
"""
SELECT *
FROM products
LIMIT ?
OFFSET ?
""";


}

```

---

Swing JTable:

```
Page 1

[Product 1-50]


Next Button


Page 2

[Product 51-100]

```

---

# 11. Better Pagination (Keyset Pagination)

OFFSET problem:

```sql
LIMIT 50 OFFSET 900000

```

Database must skip 900000 rows.

---

Better:

```sql
SELECT *

FROM products

WHERE id > 900000

LIMIT 50;

```

---

Used in:

- Large systems
    
- Infinite scrolling
    

---

# 12. Lazy Loading Concept

Problem:

Order:

```
Order

 |
 |
100 OrderItems

 |
 |
Customer

```

Loading everything:

```java
Order order =
repository.findById(1);

```

Loads:

```
Order
+
Items
+
Customer
+
Payments

```

Slow.

---

Lazy Loading:

Load only when needed.

```
Order

   |
   |
Click Items


   |
   v

Load OrderItems

```

---

# 13. N+1 Query Problem

Very common.

Code:

```java
List<Order> orders =
findAll();


for(Order order:orders){

loadItems(order);

}

```

---

SQL:

```
SELECT orders;


SELECT items WHERE order_id=1;


SELECT items WHERE order_id=2;


SELECT items WHERE order_id=3;

```

1000 orders:

1001 queries.

---

Bad performance.

---

# 14. Solution: JOIN

Instead:

```sql
SELECT

o.id,

i.product_id

FROM orders o

JOIN order_items i

ON o.id=i.order_id;

```

---

One query.

---

# 15. JDBC Batch Processing

Problem:

Insert 1000 sales.

Bad:

```java
for(Sale s:sales){

insert(s);

}

```

1000 database calls.

---

Batch:

```java
PreparedStatement ps =
connection.prepareStatement(sql);


for(Sale s:sales){

ps.addBatch();

}


ps.executeBatch();

```

---

Database calls:

```
1000

↓

1

```

---

# 16. Caching

Sometimes data rarely changes.

Example:

Products:

```
Coffee
Tea
Cake

```

Change:

Rare.

---

Without Cache:

```
Every request

     |

Database

```

---

With Cache:

```
Request

   |

Memory Cache

   |

Database only when needed

```

---

# 17. Café POS Cache Example

Product Cache:

```java
Map<Integer,Product> cache =
new ConcurrentHashMap<>();

```

---

First request:

```
Database

   |

Cache

```

---

Next:

```
Cache

   |

Return

```

---

# 18. Database Locking

Multiple cashiers:

```
Cashier A

Stock Coffee = 1


Cashier B

Stock Coffee = 1

```

Both sell.

Problem:

```
Stock becomes -1

```

---

Need locking.

---

# 19. Optimistic Locking

Use version column.

Table:

```sql
products


id

name

stock

version

```

---

Update:

```sql
UPDATE products

SET stock=?,
version=version+1

WHERE id=?

AND version=?;

```

---

If another user changed:

Update fails.

---

# 20. Pessimistic Locking

Lock row:

```sql
SELECT *

FROM products

WHERE id=1

FOR UPDATE;

```

---

Meaning:

```
User A locks row

User B waits

```

---

# 21. Transaction Isolation Levels

Database transactions have isolation levels.

## READ UNCOMMITTED

Lowest safety.

Can read dirty data.

---

## READ COMMITTED

Common.

---

## REPEATABLE READ

MySQL default.

---

## SERIALIZABLE

Highest safety.

Slow.

---

# 22. Café POS Transaction Example

Payment:

```
START TRANSACTION


Check Stock


Lock Product Row


Reduce Stock


Create Order


Commit


```

---

# 23. Deadlock

Example:

Transaction A:

```
Lock Product

wait Customer

```

Transaction B:

```
Lock Customer

wait Product

```

Both waiting.

```
DEADLOCK

```

---

# 24. Avoid Deadlock

Rules:

Always lock in same order.

Good:

```
Product

then

Order

```

Everywhere.

---

Bad:

Thread A:

```
Product

Order

```

Thread B:

```
Order

Product

```

---

# 25. Connection Pool Optimization

HikariCP settings:

Example:

```java
config.setMaximumPoolSize(20);

config.setMinimumIdle(5);

```

---

Meaning:

```
Maximum 20 DB connections

Minimum ready 5

```

---

# 26. Database Timeout

Never wait forever.

Example:

```java
statement.setQueryTimeout(
5
);

```

---

Meaning:

```
Query > 5 seconds

Cancel

```

---

# 27. PreparedStatement Performance

Good:

```java
SELECT *
FROM products
WHERE id=?

```

Database can reuse execution plan.

---

# 28. SQL Injection Prevention

Never:

```java
String sql =

"SELECT * FROM users WHERE name='"

+name+

"'";

```

---

Always:

```java
PreparedStatement

```

---

# 29. Café POS Production Database Design

Example:

```
products

 id
 name
 category_id
 price
 stock


orders

 id
 customer_id
 total
 created_at


order_items

 id
 order_id
 product_id
 quantity
 price


inventory_transactions

 id
 product_id
 type
 quantity
 created_at

```

---

Indexes:

```sql
products(name)

orders(created_at)

orders(customer_id)

order_items(order_id)

```

---

# 30. Java 25 Repository Optimization Example

Modern:

```java
public record Product(
int id,
String name,
double price
){}

```

Repository:

```java
public List<Product> findByPage(
int limit,
int offset
){

return jdbc.query(
sql,
mapper
);

}

```

---

# 31. Performance Checklist

Database:

✅ Index important columns

✅ Use EXPLAIN

✅ Pagination

✅ Transactions

✅ Connection Pool

JDBC:

✅ PreparedStatement

✅ Batch Processing

✅ Close Resources

Architecture:

✅ Repository Layer

✅ Service Layer

✅ Cache

---

# Interview Questions

## Q1: Why use Index?

To improve query performance.

---

## Q2: What is N+1 problem?

Too many database queries caused by loading related data repeatedly.

---

## Q3: Difference between optimistic and pessimistic locking?

Optimistic:

- Check version
    

Pessimistic:

- Lock row
    

---

## Q4: What is Connection Pool?

A collection of reusable database connections.

---

## Q5: Why pagination?

Avoid loading huge data sets.

---

# Practice Project

Optimize Café POS Database Layer:

Implement:

```
ProductRepository

    |
    |
findPage()

findByName()

cacheProducts()

```

Add:

1. Product index
    
2. Pagination
    
3. Transaction order processing
    
4. Stock locking
    
5. Batch sales insert
    
6. HikariCP tuning
    

---

# Lesson 18 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Database Query Optimization  
✅ Index  
✅ Composite Index  
✅ EXPLAIN  
✅ Pagination  
✅ Keyset Pagination  
✅ Lazy Loading  
✅ N+1 Problem  
✅ JOIN Optimization  
✅ Batch Processing  
✅ Cache Strategy  
✅ Database Locking  
✅ Isolation Levels  
✅ Deadlock Prevention  
✅ Connection Pool Tuning

---

# Next Lesson

# Lesson 19: Enterprise Exception Handling & Logging Architecture

## Building Production Error Management System for Café POS

Topics:

- Global Exception Handler
    
- Custom Error Codes
    
- Error Response Design
    
- Logging Architecture
    
- SLF4J + Logback
    
- Debugging Strategy
    
- Audit Logging
    
- Production Error Monitoring
    

ဒီ Lesson က သင်အရင်လေ့လာခဲ့တဲ့ **Custom Exception + Global Handler** ကို Enterprise Level အထိ တိုးချဲ့ပါမယ်။