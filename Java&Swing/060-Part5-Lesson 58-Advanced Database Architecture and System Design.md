# Part 5: Café POS System Architecture Phase

# Lesson 58: Advanced Database Architecture & System Design

## Senior Database Engineer / System Architect Level

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson ကနေ Application Developer Level ကနေ **Database Engineer / System Architect Level** ကို တက်သွားပါမယ်။

အခုအထိ Café POS System ကို:

```text
Java Code Level

        ↓

MVC Architecture

        ↓

JDBC

        ↓

MySQL CRUD

```

အထိ တည်ဆောက်ခဲ့ပါတယ်။

ဒါပေမယ့် Enterprise Application တစ်ခုမှာ User များလာတဲ့အခါ:

```text
1 user
      ↓
10 users
      ↓
100 users
      ↓
1000 users

```

ဆိုရင် Database Design မကောင်းရင် System Slow ဖြစ်သွားနိုင်ပါတယ်။

---

# Lesson 58 Goals

ဒီ Lesson မှာ လေ့လာမယ့်အရာများ:

```text
1. Database Architecture

2. Normalization vs Denormalization

3. Index Strategy

4. Query Optimization

5. Transaction Design

6. Concurrency Control

7. Locking Strategy

8. Enterprise POS Database Design

```

---

# PART 1

# Database Architecture Overview

---

# 1. Basic Database Architecture

Beginner Application:

```text
Swing App


    |

    |

 JDBC


    |

    |

 MySQL Database

```

Problem:

```text
Small Cafe

1-5 users

OK

```

---

Enterprise:

```text
                Client Layer


        Swing POS Terminal 1

        Swing POS Terminal 2

        Admin Dashboard


                 |


          Application Server


                 |


          Database Layer


                 |


              MySQL


```

---

# 2. Three Tier Architecture

Professional:

```text
+----------------------------+

Presentation Layer


Swing UI


+----------------------------+

Business Layer


Services

Rules

Transactions


+----------------------------+

Data Layer


Repository

JDBC

Database


+----------------------------+

```

---

Café POS:

```text
Swing

   ↓

Controller

   ↓

Service

   ↓

Repository

   ↓

MySQL

```

---

# PART 2

# Database Normalization

---

# 3. What is Normalization?

Normalization ဆိုတာ:

> Data duplication လျှော့ချပြီး Data Consistency တိုးစေတဲ့ Database Design Technique

---

Example Wrong Design:

Order Table:

```text
orders


id

customer_name

customer_phone

product_name

product_price


```

Problem:

Order တိုင်းမှာ:

```text
Coffee Latte

Price

Customer Info

```

ထပ်နေပါတယ်။

---

# 4. Normalized Design

Separate:

## Customers

```text
customers


id

name

phone

```

---

## Products

```text
products


id

name

price

```

---

## Orders

```text
orders


id

customer_id

date

```

---

## Order Items

```text
order_items


id

order_id

product_id

qty

price

```

---

Relationship:

```text
Customer


   1


   |


   *


Order


   1


   |


   *


Order_Item


   *


   |


   1


Product

```

---

# 5. Normalization Levels

## First Normal Form (1NF)

Rule:

```text
One column = One value

```

Wrong:

```text
order_items


products:

Latte,Coffee,Cake

```

---

Correct:

```text
order_items


Latte


Coffee


Cake

```

---

# Second Normal Form (2NF)

Remove partial dependency.

Example:

Wrong:

```text
order_item


order_id

product_id

product_name

```

Problem:

product_name depends on product_id only.

Move:

```text
products


product_id

product_name

```

---

# Third Normal Form (3NF)

Remove transitive dependency.

Wrong:

```text
customers


id

name

city

city_code

```

Separate:

```text
cities


city_code

city_name

```

---

# PART 3

# Café POS Database Normalized Design

Final:

```text
customers


        |

        |

orders


        |

        |

order_items


        |

        |

menu_items


        |

        |

recipes


        |

        |

ingredients


        |

        |

inventory


        |

        |

stock_movements

```

---

# PART 4

# Index Strategy

---

# 6. What is Index?

Index ဆိုတာ Database ရဲ့ Search Speed မြှင့်ပေးတဲ့ Data Structure ဖြစ်ပါတယ်။

Book Example:

Without Index:

```text
Search word

↓

Read every page

↓

Slow

```

---

With Index:

```text
Index Page

↓

Jump directly

↓

Fast

```

---

# 7. Without Index

Example:

```sql
SELECT *

FROM customers

WHERE phone='091234567';

```

Database:

```text
1 million rows

scan all

```

Slow.

---

# 8. With Index

Create:

```sql
CREATE INDEX idx_customer_phone

ON customers(phone);

```

Now:

```text
phone lookup

milliseconds

```

---

# 9. Important Café POS Indexes

## Customers

```sql
CREATE INDEX idx_customer_phone

ON customers(phone);

```

---

## Products

```sql
CREATE INDEX idx_product_name

ON products(name);

```

---

## Orders Date

```sql
CREATE INDEX idx_order_date

ON orders(created_at);

```

---

## Inventory

```sql
CREATE INDEX idx_inventory_ingredient

ON inventory(ingredient_id);

```

---

# PART 5

# Composite Index

---

# 10. Multiple Column Search

Example:

```sql
SELECT *

FROM orders

WHERE

customer_id=10

AND

created_at='2026-07-31';

```

---

Create:

```sql
CREATE INDEX idx_order_customer_date

ON orders(

customer_id,

created_at

);

```

---

# Rule:

Order matters:

```text
customer_id

then

created_at

```

---

# PART 6

# Query Optimization

---

# 11. Bad Query

```sql
SELECT *

FROM orders;

```

Problem:

```text
1 million rows

Load all data

```

---

Better:

```sql
SELECT

id,

total

FROM orders

LIMIT 100;

```

---

# 12. Avoid SELECT *

Bad:

```sql
SELECT *

FROM products;

```

Good:

```sql
SELECT

id,

name,

price

FROM products;

```

---

Why?

Because:

```text
Less Data

↓

Less Memory

↓

Faster

```

---

# 13. Explain Query Plan

MySQL:

```sql
EXPLAIN

SELECT *

FROM orders

WHERE customer_id=10;

```

Shows:

```text
Index used?

Rows scanned?

Cost?

```

---

# PART 7

# Transaction Architecture

---

# 14. ACID Principle

Enterprise Database uses:

## A - Atomicity

Either:

```text
Everything Success

```

or:

```text
Everything Rollback

```

---

Example:

Order:

```text
Save Order

+

Deduct Stock

+

Payment

```

All succeed or fail.

---

## C - Consistency

Database rules always valid.

Example:

Stock:

```text
Cannot become -10

```

---

## I - Isolation

Multiple users don't interfere.

---

## D - Durability

After commit:

```text
Data survives restart

```

---

# PART 8

# Café POS Transaction Example

Customer buys:

```text
Latte x2

```

Transaction:

```sql
BEGIN;


INSERT order;


INSERT order_items;


UPDATE inventory;


INSERT stock_movement;


INSERT payment;


COMMIT;

```

---

Failure:

```sql
ROLLBACK;

```

---

# PART 9

# Concurrency Control

---

# 15. Real Problem

Two Cashiers:

Cashier A:

```text
Coffee Bean Stock = 10

Sell 8

```

Cashier B:

```text
Coffee Bean Stock = 10

Sell 5

```

Problem:

```text
Stock becomes negative

```

---

# 16. Solution: Row Lock

Use:

```sql
SELECT *

FROM inventory

WHERE ingredient_id=1

FOR UPDATE;

```

---

Meaning:

```text
Lock this row

Other users wait

```

---

# 17. JDBC Example

```java
con.setAutoCommit(false);



PreparedStatement ps =

con.prepareStatement(

"""
SELECT quantity

FROM inventory

WHERE ingredient_id=?

FOR UPDATE

"""

);

```

---

# PART 10

# Database Backup Strategy

---

Professional Café:

Daily:

```text
02:00 AM

Automatic Backup

```

---

Backup:

```bash
mysqldump

cafe_pos_db

>

backup.sql

```

---

# PART 11

# Final Enterprise Database Architecture

```text
                 Café POS


                    |

              Application Layer


                    |

              Service Layer


                    |

             Repository Layer


                    |

             Transaction Manager


                    |

              MySQL Database


                    |

        -------------------------


        Indexes

        Constraints

        Backup

        Security

        Monitoring

```

---

# Current Café POS Level

After Lesson 58:

```text
Java 25 Architecture              ✅

MVC Design                        ✅

Database Normalization             ✅

Index Strategy                     ✅

Query Optimization                 ✅

ACID Transaction                   ✅

Concurrency Control                ✅

Enterprise Database Design         ✅

```

---

# Practice Task

Design:

1. Indexes for:
    

```
orders
products
inventory
stock_movements
```

2. Analyze:
    

```
EXPLAIN SELECT
```

3. Add transaction handling for:
    

```
Order + Payment + Inventory
```

---

# Next Lesson

# Lesson 59: Enterprise Security Architecture

## Authentication + Authorization + Database Security

Next we build:

```text
User Management

        ↓

Role Based Access Control (RBAC)

        ↓

Password Hashing

        ↓

Permission System

        ↓

Audit Logging

```

ပြီးရင် Café POS က **Enterprise Software Security Level** ကို ဝင်သွားပါမယ်။