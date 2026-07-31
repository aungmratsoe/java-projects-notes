# Part 2: Advanced Java Knowledge for Swing

# Lesson 22: Building Café POS Database Schema + Entity Design

## Professional ERD → MySQL → Java Model Mapping

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson ကနေစပြီး **Café POS System ကို Design Phase ကနေ Development Phase သို့ စတင်ပါမယ်။**

အခုအထိ ကျွန်တော်တို့ လေ့လာခဲ့တာ:

- Exception Architecture
    
- Logging
    
- JDBC
    
- Transaction
    
- Multithreading
    
- MVC Architecture
    

တွေကို အသုံးချပြီး **Real Enterprise Café POS Database Design** တည်ဆောက်ပါမယ်။

---

# 1. Database Design ဆိုတာဘာလဲ?

Database Design ဆိုတာ:

> Application အတွက် data တွေကို ဘယ်လိုသိမ်းမလဲ၊ ဘယ်လိုဆက်သွယ်မလဲ ဆိုတာကို ကြိုတင်တည်ဆောက်ခြင်း ဖြစ်ပါတယ်။

---

Example:

Café မှာ:

```text
Product

Coffee
Cake
Tea


Order

Customer buys products


Payment

Cash/Card


Inventory

Stock tracking

```

ဒါတွေကို Database ထဲမှာ structure တစ်ခုနဲ့ သိမ်းရပါတယ်။

---

# 2. Bad Database Design Example

Beginner:

```
sales_table


id

customer_name

product_name

quantity

price

payment_type

```

---

Problem:

## Duplicate Data

Example:

```
Order 1

Coffee
Customer: John


Order 2

Coffee
Customer: John


```

Data ထပ်နေပါတယ်။

---

## Difficult Update

Coffee price ပြောင်းရင်:

1000 rows update လုပ်ရမယ်။

---

Solution:

Normalization.

---

# 3. Database Normalization

Normalization ဆိုတာ:

> Duplicate data လျှော့ပြီး data consistency ရအောင် table ခွဲခြင်း ဖြစ်ပါတယ်။

---

# First Normal Form (1NF)

Rule:

One column = One value

Bad:

```
order_id

products

1

Coffee,Cake,Tea

```

---

Good:

```
order_items


order_id

product_id


1

10


1

20


1

30

```

---

# Second Normal Form (2NF)

Partial dependency မဖြစ်ရ။

---

Example:

Order Item:

```
order_id

product_id

product_name

quantity

```

Problem:

product_name က product_id ပေါ်ပဲမူတည်တယ်။

---

Separate:

```
products


id

name


```

```
order_items


order_id

product_id

quantity

```

---

# Third Normal Form (3NF)

Non-key dependency မရှိရ။

---

Example:

Customer:

```
customer_id

customer_name

city

city_code

```

---

Better:

customers:

```
customer_id

customer_name

city_id

```

cities:

```
city_id

city_name

```

---

# 4. Café POS Main Entities

Our system entities:

```text
User

Customer

Category

Product

Recipe

Ingredient

Inventory

Order

OrderItem

Payment

Supplier

Purchase

AuditLog

```

---

# 5. Complete Relationship Overview

```
User

 |
 |
Orders

 |
 |
OrderItems

 |
 |
Products

 |
 |
ProductRecipe

 |
 |
Ingredients

 |
 |
Inventory


```

---

# 6. Database Naming Convention

Professional naming:

Use:

snake_case

Good:

```
order_items

product_recipes

created_at

user_id

```

Avoid:

```
OrderItems

ProductRecipes

```

---

# 7. Database Creation

Database:

```sql
CREATE DATABASE cafe_pos;

USE cafe_pos;

```

---

# 8. Users Table

Purpose:

Cashier / Admin login.

```sql
CREATE TABLE users(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

username VARCHAR(50)
NOT NULL UNIQUE,

password_hash VARCHAR(255)
NOT NULL,

role VARCHAR(30)
NOT NULL,

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

Example data:

```
id | username | role

1  | admin    | ADMIN

2  | john     | CASHIER

```

---

# 9. Category Table

Example:

Coffee

Food

Dessert

```sql
CREATE TABLE categories(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100)
NOT NULL UNIQUE

);

```

---

Relationship:

```
Category

    |

    |

 Products

```

---

# 10. Product Table

Important table.

```sql
CREATE TABLE products(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


category_id BIGINT NOT NULL,


name VARCHAR(100)
NOT NULL,


price DECIMAL(10,2)
NOT NULL,


status BOOLEAN DEFAULT TRUE,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(category_id)

REFERENCES categories(id)

);

```

---

Relationship:

```
Category 1

   |

   |

Many Products

```

---

# 11. Ingredient Table

Example:

```
Coffee Bean

Milk

Sugar

Chocolate

```

---

Table:

```sql
CREATE TABLE ingredients(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


name VARCHAR(100)
NOT NULL,


unit VARCHAR(20)

);

```

---

# 12. Product Recipe Table

This is important.

You requested earlier:

`product_recipes`

Example:

Coffee:

```
Coffee

 |

 |-- Coffee Bean 20g

 |-- Milk 100ml

 |-- Sugar 5g

```

---

Database:

```sql
CREATE TABLE product_recipes(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


product_id BIGINT NOT NULL,


ingredient_id BIGINT NOT NULL,


quantity DECIMAL(10,2)
NOT NULL,


FOREIGN KEY(product_id)

REFERENCES products(id),


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

Relationship:

```
Product

  |

  |

Product Recipe

  |

  |

Ingredient

```

---

# 13. Inventory Table

Track stock.

```sql
CREATE TABLE inventory(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


ingredient_id BIGINT NOT NULL,


quantity DECIMAL(10,2)
DEFAULT 0,


updated_at TIMESTAMP
DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

Example:

```
Coffee Bean

Stock:

5000 gram

```

---

# 14. Customer Table

```sql
CREATE TABLE customers(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


name VARCHAR(100),


phone VARCHAR(30),


created_at TIMESTAMP
DEFAULT CURRENT_TIMESTAMP

);

```

---

# 15. Orders Table

Order Header.

```sql
CREATE TABLE orders(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


customer_id BIGINT,


user_id BIGINT NOT NULL,


total_amount DECIMAL(10,2),


status VARCHAR(30),


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(customer_id)

REFERENCES customers(id),


FOREIGN KEY(user_id)

REFERENCES users(id)

);

```

---

# 16. Order Items Table

Products inside order.

```sql
CREATE TABLE order_items(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


order_id BIGINT NOT NULL,


product_id BIGINT NOT NULL,


quantity INT NOT NULL,


price DECIMAL(10,2)
NOT NULL,


FOREIGN KEY(order_id)

REFERENCES orders(id),


FOREIGN KEY(product_id)

REFERENCES products(id)

);

```

---

Example:

Order:

```
Order #100

```

Items:

```
Coffee x2

Cake x1

```

---

# 17. Payment Table

```sql
CREATE TABLE payments(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


order_id BIGINT NOT NULL,


payment_method VARCHAR(30),


amount DECIMAL(10,2),


paid_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(order_id)

REFERENCES orders(id)

);

```

---

# 18. Audit Log Table

Security tracking.

```sql
CREATE TABLE audit_logs(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


user_id BIGINT,


action VARCHAR(100),


description TEXT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(user_id)

REFERENCES users(id)

);

```

---

# 19. Complete ERD

High-level:

```
                 USERS
                   |
                   |
                 ORDERS
                   |
              ORDER_ITEMS
                   |
                   |
                PRODUCTS
                   |
        --------------------
        |                  |
   CATEGORY        PRODUCT_RECIPES
                              |
                              |
                         INGREDIENTS
                              |
                              |
                         INVENTORY


ORDERS

   |

PAYMENTS


```

---

# 20. Java Entity Mapping

Database:

```
products

id

name

price

```

---

Java 25 Record:

```java
public record Product(

Long id,

String name,

BigDecimal price

){

}

```

---

Why BigDecimal?

Money calculation.

Wrong:

```java
double price;

```

Problem:

```
0.1 + 0.2

=

0.3000000004

```

---

Correct:

```java
BigDecimal

```

---

# 21. Entity Relationship Mapping

Product:

```java
public record Product(

Long id,

Category category,

String name,

BigDecimal price

){

}

```

---

Order:

```java
public record Order(

Long id,

Customer customer,

List<OrderItem> items,

BigDecimal total

){

}

```

---

# 22. DTO Design

Swing does not need full Entity.

Create:

```java
public record ProductDTO(

String name,

BigDecimal price

){

}

```

---

Flow:

```
JTable

 |

DTO

 |

Service

 |

Entity

 |

Database

```

---

# 23. Database Layer Structure

Final:

```
database

 |
 |
DatabaseManager


model

 |
 |
Product
Order
Customer


repository

 |
 |
ProductRepository


repository.impl

 |
 |
MySQLProductRepository

```

---

# 24. Database Rules

Important:

## Foreign Keys

Always.

Example:

```
order_items

must have existing order

```

---

## Constraints

Use:

```
NOT NULL

UNIQUE

CHECK

FOREIGN KEY

```

---

## Soft Delete

Instead of:

```sql
DELETE FROM products;

```

Use:

```sql
UPDATE products

SET status=false;

```

---

Why?

Keep history.

---

# 25. Café POS Transaction Example

Customer checkout:

```
BEGIN


Insert Order


Insert Order Items


Update Inventory


Insert Payment


Insert Audit Log


COMMIT


```

---

If error:

```
ROLLBACK

```

---

# 26. Professional Database Checklist

Our design has:

✅ Normalization

✅ PK/FK

✅ Inventory System

✅ Recipe System

✅ Order Management

✅ Payment Tracking

✅ Audit Logging

✅ Transaction Support

---

# Practice Task

Create:

```
cafe_pos.sql

```

Containing:

1. Database creation
    
2. 10 tables
    
3. Foreign Keys
    
4. Indexes
    

Next we will connect this database with Java.

---

# Lesson 22 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Database Design Concept  
✅ Normalization  
✅ Entity Design  
✅ Café POS ERD  
✅ Product Recipe System  
✅ Inventory Model  
✅ Order Model  
✅ Payment Model  
✅ Audit Log  
✅ Java Record Mapping  
✅ BigDecimal Money Handling  
✅ Enterprise Database Structure

---

# Next Lesson

# Lesson 23: JDBC Implementation for Café POS

## Building Repository Layer with MySQL

Topics:

- DatabaseManager
    
- HikariCP Setup
    
- Repository Interface
    
- MySQL Repository Implementation
    
- CRUD Operations
    
- Transaction Service
    
- Exception Integration
    

ဒီ Lesson မှာ Database Design ကို **တကယ် Java Code နဲ့ ချိတ်ပြီး Build စတင်ပါမယ်။**