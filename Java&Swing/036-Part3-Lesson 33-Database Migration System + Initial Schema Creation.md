# Part 3: Café POS Real Implementation Phase

# Lesson 33: Database Migration System + Initial Schema Creation

## Creating Professional Café POS MySQL Database Automatically

### (Java 25 + JDBC + MySQL + Migration Architecture)

ဒီ Lesson မှာ Café POS ရဲ့ **Database Foundation ကို Production Level** အဖြစ် တည်ဆောက်ပါမယ်။

အခုအထိ:

```
Java Application
        |
Database Layer
        |
HikariCP
        |
MySQL Connection
```

ရပြီးပါပြီ။

ဒီနေ့မှာ:

```
Empty MySQL Database

        ↓

Migration Runner

        ↓

V1__init.sql

        ↓

Complete Café POS Schema

```

ဖြစ်အောင်လုပ်ပါမယ်။

---

# 1. Why Database Migration System?

Beginner Approach:

Database ကို manual create:

```
Open MySQL Workbench

Copy SQL

Run

```

Problem:

- Team member အသစ် join လုပ်ရင်?
    
- Production server အသစ်တင်ရင်?
    
- Database version ပြောင်းရင်?
    

ခက်ပါတယ်။

---

# Professional Approach

Migration:

```
Application Start

       |

Check Database Version

       |

Run Missing SQL

       |

Database Updated

```

---

# 2. Migration Architecture

Create:

```
database

├── MigrationManager.java

├── MigrationRepository.java

└── migration


resources

└── database

      └── migration

            └── V1__init.sql

```

---

# 3. Migration Table

Database ထဲမှာ အရင်ဆုံး:

```sql
schema_version

```

ရှိရမယ်။

Purpose:

```
Which SQL already executed?

```

---

Table:

```sql
CREATE TABLE schema_version(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

version VARCHAR(50) NOT NULL,

description VARCHAR(255),

executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

Example Data:

```
id | version | description

1     V1       Initial Schema

2     V2       Add Inventory

```

---

# 4. Migration File Naming Convention

Professional style:

```
V1__init.sql


V2__inventory.sql


V3__reporting.sql

```

Format:

```
V{number}__{description}.sql

```

---

# 5. Create Migration Folder

Inside:

```
src/main/resources


database


└── migration


    └── V1__init.sql

```

---

# 6. V1 Initial Database Schema

Now create:

```
V1__init.sql

```

---

## 6.1 Roles Table

```sql
CREATE TABLE roles(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(50) UNIQUE NOT NULL

);

```

---

Insert Default Roles:

```sql
INSERT INTO roles(name)

VALUES

('ADMIN'),

('MANAGER'),

('CASHIER');

```

---

# 6.2 Users Table

```sql
CREATE TABLE users(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

username VARCHAR(50)
UNIQUE NOT NULL,

password_hash VARCHAR(255)
NOT NULL,

role_id BIGINT NOT NULL,

active BOOLEAN DEFAULT TRUE,

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


CONSTRAINT fk_user_role

FOREIGN KEY(role_id)

REFERENCES roles(id)

);

```

---

Relationship:

```
Role

1
|
|
Many

Users

```

---

# 6.3 Category Table

Products need category.

Example:

```
Coffee

Food

Dessert

Drink

```

---

SQL:

```sql
CREATE TABLE categories(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100)
UNIQUE NOT NULL

);

```

---

# 6.4 Product Table

```sql
CREATE TABLE products(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

category_id BIGINT,

name VARCHAR(100)
NOT NULL,

price DECIMAL(10,2)
NOT NULL,

active BOOLEAN DEFAULT TRUE,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(category_id)

REFERENCES categories(id)

);

```

---

Example:

```
Coffee Latte

5000 MMK

```

---

# 6.5 Ingredients Table

Inventory uses ingredients.

```sql
CREATE TABLE ingredients(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100)
NOT NULL,

unit VARCHAR(20)
NOT NULL

);

```

---

Example:

```
Coffee Bean

gram


Milk

ml

```

---

# 6.6 Inventory Table

Current stock:

```sql
CREATE TABLE inventory(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


ingredient_id BIGINT UNIQUE,


quantity DECIMAL(10,2)
DEFAULT 0,


updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

# 6.7 Product Recipe Table

Very important.

Example:

```
Coffee Latte


Coffee Bean 20g

Milk 100ml

Sugar 5g

```

---

SQL:

```sql
CREATE TABLE product_recipes(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


product_id BIGINT,


ingredient_id BIGINT,


quantity DECIMAL(10,2),


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

Recipe

  |

Ingredient

```

---

# 6.8 Orders Table

```sql
CREATE TABLE orders(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


user_id BIGINT,


total_amount DECIMAL(10,2),


status VARCHAR(30),


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(user_id)

REFERENCES users(id)

);

```

---

Status:

```
PENDING

PAID

CANCELLED

```

---

# 6.9 Order Items Table

```sql
CREATE TABLE order_items(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


order_id BIGINT,


product_id BIGINT,


quantity INT,


price DECIMAL(10,2),



FOREIGN KEY(order_id)

REFERENCES orders(id),


FOREIGN KEY(product_id)

REFERENCES products(id)

);

```

---

# 6.10 Payments Table

```sql
CREATE TABLE payments(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


order_id BIGINT,


method VARCHAR(30),


amount DECIMAL(10,2),


paid_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(order_id)

REFERENCES orders(id)

);

```

---

Payment:

```
CASH

CARD

KBZ_PAY

WAVE_PAY

```

---

# 6.11 Stock Movement Table

```sql
CREATE TABLE stock_movements(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


ingredient_id BIGINT,


type VARCHAR(30),


quantity DECIMAL(10,2),


reference_id BIGINT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

# 6.12 Audit Log Table

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

# 7. Database Index Optimization

Large POS:

```
Orders = millions of rows

```

Need Index.

---

Orders:

```sql
CREATE INDEX idx_orders_date

ON orders(created_at);

```

---

Products:

```sql
CREATE INDEX idx_product_name

ON products(name);

```

---

Order Items:

```sql
CREATE INDEX idx_order_items_order

ON order_items(order_id);

```

---

# 8. MigrationManager Design

Class:

```
MigrationManager.java

```

Purpose:

```
Read SQL Files

Execute SQL

Save Version

```

---

Structure:

```java
public class MigrationManager {


public void migrate(){


}


}

```

---

# 9. Migration Flow

```
Application Start

        |

MigrationManager.migrate()

        |

Find SQL Files

        |

Check schema_version

        |

Execute New SQL

        |

Save Version


```

---

# 10. Application Startup Order

Important:

Wrong:

```
Start UI

Connect Database

Run Migration

```

---

Correct:

```
Application.main()


       |

Load Configuration


       |

Initialize Database


       |

Run Migration


       |

Initialize Security


       |

Open Login Screen


```

---

# 11. Final Database ERD

High Level:

```
              roles

                |

                |

              users


                |

                |

              orders

              /    \

             /      \


      order_items   payments


             |

             |

          products

             |

             |

       product_recipes

             |

             |

        ingredients

             |

             |

        inventory


```

---

# 12. Current Project Status

Completed:

```
Project Setup          ✅

Java 25                ✅

Maven                  ✅

Swing Foundation       ✅

Database Connection    ✅

HikariCP               ✅

Transaction Manager    ✅

Database Schema Design ✅

Migration Design       ✅

```

---

# Practice Task

Create:

1. `V1__init.sql`
    
2. Create MySQL database:
    

```
cafe_pos
```

3. Run migration manually
    
4. Verify tables:
    

```sql
SHOW TABLES;

```

Expected:

```
roles

users

products

categories

ingredients

inventory

product_recipes

orders

order_items

payments

stock_movements

audit_logs

schema_version

```

---

# Next Lesson

# Lesson 34: Building Authentication Module

## User Entity + BCrypt + Login Swing UI + Session Management

Next we will implement the first complete feature:

```
Database

↓

User Repository

↓

Authentication Service

↓

Login Controller

↓

Professional Swing Login Screen

↓

Main Dashboard

```

ဒီ Lesson ပြီးရင် Café POS ကို **တကယ် Login ဝင်ပြီး အသုံးပြုနိုင်တဲ့ Application** ဖြစ်လာပါမယ်။